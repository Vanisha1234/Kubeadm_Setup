<h1 align="center">🚀 Kubeadm Setup from Scratch</h1>

<p align="center">
A hands-on DevOps project demonstrating how to build a multi-node Kubernetes cluster from scratch using <b>Vagrant</b>, <b>VirtualBox</b>, and <b>Kubeadm</b>.  
This setup closely simulates a real-world production-like environment, showcasing my understanding of Kubernetes fundamentals, networking (CNI), and cluster provisioning.
</p>

---

## 🧩 Project Overview

In this project, I’ve automated the provisioning of a Kubernetes cluster consisting of **one Control Plane** and **two Worker Nodes** using **Vagrant** and **VirtualBox**.  

Since the official Kubernetes documentation can be complex sometimes, I’ve summarized all required steps into one simplified, beginner friendly guide.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|----------|
| 🪟 **Windows 10** | Host Operating System |
| 🧱 **VirtualBox** | Virtualization Provider |
| ⚙️ **Vagrant** | VM Automation |
| ☸️ **Kubernetes (kubeadm)** | Cluster Orchestration |
| 🌐 **Flannel** | CNI Plugin for Pod Networking |

---

## ⚙️ Setting up VirtualBox & Vagrant

### Step 1: Install Dependencies

Install the following tools before proceeding:

- [🔗 Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [🔗 Download Vagrant](https://developer.hashicorp.com/vagrant/install)


### Step 2: Clone the KodeKloud Vagrant Repository

```bash
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
```
This repo contains a Vagrantfile capable of spinning up:

🧠 1 Control Plane Node (controlplane) \
🧩 2 Worker Nodes (node01, node02)

Navigate to the location where Vagrantfile is present:
```bash
C:\Users\vanis\certified-kubernetes-administrator-course\kubeadm-clusters\virtualbox\Vagrantfile
```
Run command:
```bash
vd@VanishaDeb:~/certified-kubernetes-administrator-course/kubeadm-clusters/virtualbox$ vagrant status
Current machine states:

controlplane              not created (virtualbox)
node01                    not created (virtualbox)
node02                    not created (virtualbox)
```
➡️ This confirms the VMs are defined but not yet created.

To spin up all three virtual machines defined in Vagrantfile: \
Run command:
```bash
Vagrant up
```
Output:
```bash
PS C:\Users\vanis\certified-kubernetes-administrator-course\kubeadm-clusters\virtualbox> vagrant up
Bringing machine 'controlplane' up with 'virtualbox' provider...
Bringing machine 'node01' up with 'virtualbox' provider...
Bringing machine 'node02' up with 'virtualbox' provider...
==> controlplane: Importing base box 'ubuntu/jammy64'...
==> controlplane: Matching MAC address for NAT networking...
==> controlplane: Setting the name of the VM: controlplane
==> controlplane: Clearing any previously set network interfaces...
==> controlplane: Preparing network interfaces based on configuration...
    controlplane: Adapter 1: nat
    controlplane: Adapter 2: bridged
==> controlplane: Forwarding ports...
    controlplane: 22 (guest) => 2222 (host) (adapter 1)
==> controlplane: Running 'pre-boot' VM customizations...
==> controlplane: Booting VM...
==> controlplane: Waiting for machine to boot. This may take a few minutes...
    controlplane: SSH address: 127.0.0.1:2222
    controlplane: SSH username: vagrant
    controlplane: SSH auth method: private key
    controlplane: Warning: Connection reset. Retrying...
    controlplane: Warning: Connection aborted. Retrying...
    controlplane: 
    controlplane: Vagrant insecure key detected. Vagrant will automatically replace
    controlplane: this with a newly generated keypair for better security.
    controlplane: 
    controlplane: Inserting generated public key within guest...
    controlplane: Removing insecure key from the guest if it's present...
    controlplane: Key inserted! Disconnecting and reconnecting using new SSH key...
==> controlplane: Machine booted and ready!
==> controlplane: Checking for guest additions in VM...
    controlplane: The guest additions on this VM do not match the installed version of
    controlplane: VirtualBox! In most cases this is fine, but in rare cases it can
    controlplane: prevent things such as shared folders from working properly. If you see
    controlplane: shared folder errors, please make sure the guest additions within the
    controlplane: virtual machine match the version of VirtualBox you have installed on
    controlplane: your host and reload your VM.
    controlplane:
    controlplane: Guest Additions Version: 6.0.0 r127566
    controlplane: VirtualBox Version: 7.2
==> controlplane: Setting hostname...
==> controlplane: Configuring and enabling network interfaces...
==> controlplane: Mounting shared folders...
    controlplane: C:/Users/vanis/certified-kubernetes-administrator-course/kubeadm-clusters/virtualbox => /vagrant
==> controlplane: Running provisioner: setup-hosts (shell)...
    controlplane: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-qz38wb.sh
==> controlplane: Running provisioner: setup-dns (shell)...
    controlplane: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-2s8epz.sh
==> controlplane: Running provisioner: setup-ssh (shell)...
    controlplane: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-sl939.sh
==> controlplane: Running provisioner: file...
    controlplane: ./ubuntu/tmux.conf => $HOME/.tmux.conf
==> controlplane: Running provisioner: file...
    controlplane: ./ubuntu/vimrc => $HOME/.vimrc
==> controlplane: Running action triggers after up ...
==> controlplane: Running trigger: Post provisioner...
    Nothing to do here
==> node01: Importing base box 'ubuntu/jammy64'...
==> node01: Matching MAC address for NAT networking...
==> node01: Setting the name of the VM: node01
==> node01: Fixed port collision for 22 => 2222. Now on port 2200.
==> node01: Clearing any previously set network interfaces...
==> node01: Preparing network interfaces based on configuration...
    node01: Adapter 1: nat
    node01: Adapter 2: bridged
==> node01: Forwarding ports...
    node01: 22 (guest) => 2200 (host) (adapter 1)
==> node01: Running 'pre-boot' VM customizations...
==> node01: Booting VM...
==> node01: Waiting for machine to boot. This may take a few minutes...
    node01: SSH address: 127.0.0.1:2200
    node01: SSH username: vagrant
    node01: SSH auth method: private key
    node01: Warning: Connection reset. Retrying...
    node01: Warning: Connection aborted. Retrying...
    node01: Warning: Connection reset. Retrying...
    node01: Warning: Connection aborted. Retrying...
    node01: 
    node01: Vagrant insecure key detected. Vagrant will automatically replace
    node01: this with a newly generated keypair for better security.
    node01: 
    node01: Inserting generated public key within guest...
    node01: Removing insecure key from the guest if it's present...
    node01: Key inserted! Disconnecting and reconnecting using new SSH key...
==> node01: Machine booted and ready!
==> node01: Checking for guest additions in VM...
    node01: The guest additions on this VM do not match the installed version of
    node01: VirtualBox! In most cases this is fine, but in rare cases it can
    node01: prevent things such as shared folders from working properly. If you see
    node01: shared folder errors, please make sure the guest additions within the
    node01: virtual machine match the version of VirtualBox you have installed on
    node01: your host and reload your VM.
    node01:
    node01: Guest Additions Version: 6.0.0 r127566
    node01: VirtualBox Version: 7.2
==> node01: Setting hostname...
==> node01: Configuring and enabling network interfaces...
==> node01: Mounting shared folders...
    node01: C:/Users/vanis/certified-kubernetes-administrator-course/kubeadm-clusters/virtualbox => /vagrant
==> node01: Running provisioner: setup-hosts (shell)...
    node01: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-d5zan0.sh
==> node01: Running provisioner: setup-dns (shell)...
    node01: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-tqntgp.sh
==> node01: Running provisioner: setup-ssh (shell)...
    node01: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-zitydv.sh
==> node01: Running action triggers after up ...
==> node01: Running trigger: Post provisioner...
    Nothing to do here
==> node02: Importing base box 'ubuntu/jammy64'...
==> node02: Matching MAC address for NAT networking...
==> node02: Setting the name of the VM: node02
==> node02: Fixed port collision for 22 => 2222. Now on port 2201.
==> node02: Clearing any previously set network interfaces...
==> node02: Preparing network interfaces based on configuration...
    node02: Adapter 1: nat
    node02: Adapter 2: bridged
==> node02: Forwarding ports...
    node02: 22 (guest) => 2201 (host) (adapter 1)
==> node02: Running 'pre-boot' VM customizations...
==> node02: Booting VM...
==> node02: Waiting for machine to boot. This may take a few minutes...
    node02: SSH address: 127.0.0.1:2201
    node02: SSH username: vagrant
    node02: SSH auth method: private key
    node02: Warning: Connection reset. Retrying...
    node02: Warning: Connection aborted. Retrying...
    node02: 
    node02: Vagrant insecure key detected. Vagrant will automatically replace
    node02: this with a newly generated keypair for better security.
    node02: 
    node02: Inserting generated public key within guest...
    node02: Removing insecure key from the guest if it's present...
    node02: Key inserted! Disconnecting and reconnecting using new SSH key...
==> node02: Machine booted and ready!
==> node02: Checking for guest additions in VM...
    node02: The guest additions on this VM do not match the installed version of
    node02: VirtualBox! In most cases this is fine, but in rare cases it can
    node02: prevent things such as shared folders from working properly. If you see
    node02: shared folder errors, please make sure the guest additions within the
    node02: virtual machine match the version of VirtualBox you have installed on
    node02: your host and reload your VM.
    node02:
    node02: Guest Additions Version: 6.0.0 r127566
    node02: VirtualBox Version: 7.2
==> node02: Setting hostname...
==> node02: Configuring and enabling network interfaces...
==> node02: Mounting shared folders...
    node02: C:/Users/vanis/certified-kubernetes-administrator-course/kubeadm-clusters/virtualbox => /vagrant
==> node02: Running provisioner: setup-hosts (shell)...
    node02: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-7zhter.sh
==> node02: Running provisioner: setup-dns (shell)...
    node02: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-c36jrk.sh
==> node02: Running provisioner: setup-ssh (shell)...
    node02: Running: C:/Users/vanis/AppData/Local/Temp/vagrant-shell20250925-13792-2jb7bt.sh
==> node02: Running action triggers after up ...
==> node02: Running trigger: Post provisioner...
    Gathering IP addresses of nodes...
    Setting /etc/hosts on nodes...
Uploading hosts.tmp to /tmp/hosts.tmp
Upload has completed successfully!

  Source: hosts.tmp
  Destination: /tmp/hosts.tmp
192.168.1.26  controlplane
192.168.1.27  node01
192.168.1.28  node02
Uploading hosts.tmp to /tmp/hosts.tmp
Upload has completed successfully!

  Source: hosts.tmp
  Destination: /tmp/hosts.tmp
192.168.1.26  controlplane
192.168.1.27  node01
192.168.1.28  node02
Uploading hosts.tmp to /tmp/hosts.tmp
Upload has completed successfully!

  Source: hosts.tmp
  Destination: /tmp/hosts.tmp
192.168.1.26  controlplane
192.168.1.27  node01
192.168.1.28  node02

VM build complete!

Use either of the following to access any NodePort services you create from your browser
replacing "port_number" with the number of your NodePort.

  http://192.168.1.27:port_number
  http://192.168.1.28:port_number
```
Running VMs can be seen on Virtualbox UI as well:

<img width="1919" height="825" alt="image" src="https://github.com/user-attachments/assets/8058f45d-47a5-4028-8ac6-7605bf9d9674" />

In order to login to particular VM , run the following command:
```bash
vagrant ssh $vmname
```

---

## ☸️ Deployment with Kubeadm


  
