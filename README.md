# Vagrant setup for Cilium multi-node development

## HOWTO

### Create VMs

0. Replace path to your local Cilium dir in `Vagrantfile`.
1. `vagrant up`

### Provision K8s

0. `vagrant ssh cilium1` and `kubeadm init --skip-phases=addon/kube-proxy --apiserver-advertise-address=192.168.34.11`. Follow the instructions to copy the kubeconfig.
1.  Copy the kubeconfig to `cilium2`: `scp -r ~/.kube vagrant@192.168.34.12:`
    (password `vagrant`).
2. Join the cluster from `cilium2` node: `kubeadm join <... from the
   instructions above...>`.
3. Check that both nodes are found and have `InternalIP` set: `kubectl get nodes -o
   wide`.

### Build and install Cilium

0. You can build Cilium either on your local machine or on VM. The latter can
   be done by `vagrant ssh cilium1` and then `cd /cilium && make build`
1. Install Cilium on all VMs by `vagrant ssh cilium{1,2}` and then `cd /cilium
   && sudo make instal`
2. If you do changes only to `bpf/`, then running `sudo make install-bpf` from
   the VMs is enough.

### Run Cilium

0. Start `cilium-operator` on `cilium1`: `cd /cilium && sudo
   ./operator/cilium-operator --k8s-kubeconfig-path=/home/vagrant/.kube/config
   --cluster-pool-ipv4-cidr=10.154.0.0/16 --enable-ipv6=false`.
1. Run Cilium on both nodes: `cd /cilium && sudo sudo ./daemon/cilium-agent
   --identity-allocation-mode=crd --enable-ipv4=true
   --enable-ipv6=false --disable-envoy-version-check=true --tunnel=disabled
   --k8s-kubeconfig-path=/home/vagrant/.kube/config
   --kube-proxy-replacement=strict --ipv4-native-routing-cidr=10.154.0.0/16
   --auto-direct-node-routes=true
   --enable-bpf-masquerade=true --enable-remote-node-identity=true
   --enable-l7-proxy=false`
