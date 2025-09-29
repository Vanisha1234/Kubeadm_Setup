ISSUE:
Generally the Virtual Machines running on Virtual Box via Vagrant have dynamic IPs , which changes if a VM is restarted or while re-establishing a connection
In that case the following IP specified below while initializing control plane will change and the connection inside the cluster will break
sudo kubeadm init --apiserver-advertise-address 192.168.18.238 --pod-network-cidr "10.244.0.0/16" --upload-certs 
SOLUTION:
Follow the below steps to reset the control plan and reinitialised it:
sudo kubeadm reset -f
sudo rm -rf /etc/kubernetes/manifests
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/lib/kubelet/*
sudo rm -rf /etc/cni/net.d
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X

Reinitialize the controlplane using the current ip of the VM
sudo kubeadm init --apiserver-advertise-address 192.168.18.238 --pod-network-cidr "10.244.0.0/16" --upload-certs 

Going forward we need configure everything again including Kbectl, networking etc
Permanent fix is to assign a Fixed IP to VMS
