<h1 align="center">🧾 Errors & Solutions</h1>

# 🛠️ Fix: Control Plane Not Ready / CoreDNS Pending After Applying Flannel CNI 

If **Control Plane** stays in `NotReady` state and **CoreDNS pods remain in `Pending` state** even after applying `kube-flannel.yml`, follow this debugging and fix procedure.

## 🔍 Debugging the Issue

### 1️⃣ Inspect Kubelet Logs

```bash
sudo journalctl -u kubelet -f
```

**Common Error Output:**

```
Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized
```

### 2️⃣ Check Containerd Status

```bash
sudo systemctl status containerd --no-pager
```

**Likely Response:**

```
No cni config template is specified, wait for other system components to drop the config.
```

### 3️⃣ Verify Flannel CNI Config

```bash
cat /etc/cni/net.d/10-flannel.conflist
```

If **missing or incomplete**, kubelet cannot initialize CNI properly because required fields like `delegate.ipMasq`, `delegate.bridge`, or `subnetFile` are absent.

This results in:

```
Network plugin returns error: cni plugin not initialized
```

## ✅ Solution: Inject Correct Flannel CNI Configuration

Apply the following config:

```bash
sudo tee /etc/cni/net.d/10-flannel.conflist <<EOF
{
  "cniVersion": "0.3.1",
  "name": "cbr0",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "isDefaultGateway": true,
        "hairpinMode": true,
        "forceAddress": true,
        "ipMasq": true,
        "mtu": 1450
      }
    },
    {
      "type": "portmap",
      "capabilities": { "portMappings": true }
    }
  ]
}
EOF
```

## 🔄 Restart Services

```bash
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

✅ After restart, your **Control Plane should move to `Ready` state**, and **CoreDNS pods should start running successfully.**

```bash
kubectl get nodes
kubectl get pods -n kube-system
```
---

## ❌ Error While Registering Worker Node to Kubernetes Cluster

When running `kubeadm join` on a node, you may encounter the following error:

```bash
vagrant@node01:~$ sudo kubeadm join 192.168.1.26:6443 --token jwimhd.5siqtr1yae1sro5e \
        --discovery-token-ca-cert-hash sha256:99dab94fa8d0a3d94b76a1a40820f2e9b6e30dd925f40a349357af027df4d591

[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1

[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

### ✅ Root Cause

**IP forwarding is disabled** on the worker node. Kubernetes requires `net.ipv4.ip_forward = 1` to allow packet routing between pods and nodes.

### ✅ Solution: Enable IP Forwarding

```bash
# Open sysctl configuration
sudo nano /etc/sysctl.conf

# Uncomment or add the following line:
net.ipv4.ip_forward=1

# Save and apply configuration
sudo sysctl -p

# Verify the change
cat /proc/sys/net/ipv4/ip_forward
# Expected Output:
1
```

### 🔁 Retry Joining the Cluster

```bash
sudo kubeadm join 192.168.1.26:6443 --token <TOKEN> \
    --discovery-token-ca-cert-hash <SHA256_HASH>
```

✅ The node should now register successfully!

---

## ❌ Flannel Pods in CrashLoopBackOff / New Pods Stuck at `ContainerCreating`

If **Flannel pods keep crashing** or **other pods are stuck in `ContainerCreating` state**, it usually indicates that **network bridging is not properly enabled** on the Kubernetes nodes.

### ✅ Root Cause

Kernel modules and bridge network settings required by Flannel are **not enabled**, causing pod network initialization to fail.

### ✅ Solution: Enable Required Networking Modules

Run the following commands **on all Kubernetes nodes (Control Plane + Workers):**

```bash
# Load the br_netfilter module
sudo modprobe br_netfilter

# Enable bridge network packet filtering
sudo sysctl -w net.bridge.bridge-nf-call-iptables=1
sudo sysctl -w net.bridge.bridge-nf-call-ip6tables=1
```

### ✅ (Optional) Persist Settings Across Reboots

Add the following lines to `/etc/sysctl.conf`:

```bash
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
```

Then apply:

```bash
sudo sysctl -p
```

### 🔁 Restart Flannel

```bash
kubectl delete pod -n kube-flannel -l app=flannel - restart flannel
```

Kubernetes will recreate them automatically.

✅ After this, Flannel should stabilize and pods should start running normally.

---

# 🚀 Fixing Dynamic IP Issues in Kubernetes Cluster on Vagrant VMs

When using **Vagrant with VirtualBox**, virtual machines typically receive **dynamic IPs**. This becomes a major issue when running a **Kubernetes cluster**, especially for the **Control Plane Node**, as the API server is initialized with a specific IP.

If the VM restarts and gets a **new IP**, your cluster **breaks** because `kubeadm init` was executed using an **old IP**:

```bash
sudo kubeadm init --apiserver-advertise-address 192.168.18.238 --pod-network-cidr "10.244.0.0/16" --upload-certs
```

---

## ❌ Problem

> When the Control Plane node’s IP changes, all Kubernetes components tied to the old IP become invalid.  
> To recover, the Control Plane must be reset and reinitialized from scratch.

---

## ✅ Solution: Reset & Reinitialize Control Plane

Run the following commands **on the Control Plane VM**:

```bash
# 1. Reset the cluster
sudo kubeadm reset -f

# 2. Clean up Kubernetes directories
sudo rm -rf /etc/kubernetes/manifests
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/lib/kubelet/*
sudo rm -rf /etc/cni/net.d

# 3. Flush iptables
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X
```

Now **reinitialize the cluster** with the **current IP of the VM**:

```bash
sudo kubeadm init --apiserver-advertise-address <CURRENT_VM_IP> --pod-network-cidr "10.244.0.0/16" --upload-certs
```

---

## 🔁 Post-Reinitialization Steps

After initialization:

- Reconfigure **kubectl access**
- Reapply **CNI (Pod Network Plugin)** such as Flannel or Calico
- Rejoin worker nodes using the new `kubeadm join` token

---

## 🛡 Permanent Fix (Recommended)

To **avoid repeating this process**, assign **static IPs** to all Vagrant VMs in your `Vagrantfile`:

```ruby
config.vm.network "private_network", ip: "192.168.18.50"
```

> Setting fixed IPs ensures your Kubernetes cluster remains stable across reboots.

---

## ✅ Summary

| Issue | Cause | Temporary Fix | Permanent Fix |
|--------|------|---------------|----------------|
| Kubernetes cluster breaks after VM reboot | Dynamic IP assigned by Vagrant | Reset & reinitialize control plane | Use **Static IPs** in `Vagrantfile` |
