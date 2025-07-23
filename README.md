# MicroK8s
sudo snap install microk8s --classic --channel=1.33/stable
echo "alias k='microk8s kubectl'" >> ~/.bashrc
source ~/.bashrc

# Operator
git clone git@github.com:cbarbieru/operator.git
git checkout microk8s_patch

cd ~/operator/config/release/
k apply -k .

k label node rosablanche-1 node.kubernetes.io/worker="" --overwrite
k get node rosablanche-1 --show-labels

cd ~/operator/config/samples/ccruntime/default
k apply -k .

sudo vim /var/snap/microk8s/current/args/config.toml.d/nydus-snapshotter.toml

[proxy_plugins]
  [proxy_plugins.nydus]
	type = "snapshot"
	address = "/run/containerd-nydus/containerd-nydus-grpc.sock"

sudo systemctl start nydus-snapshotter.service

sudo vim /var/snap/microk8s/current/args/containerd-template.toml

imports = ["/var/snap/microk8s/current/args/config.toml.d/*.toml"]


sudo vim /opt/kata/share/defaults/kata-containers/configuration-qemu-tdx.toml

[hypervisor.qemu]
path = "/opt/kata/bin/qemu-system-x86_64-tdx-experimental"
# confidential_guest = true
time="2025-07-23T12:24:05.013714495Z" level=error msg="qemu-system-x86_64-tdx-experimental: vm-type TDX not supported by KVM" name=containerd-shim-v2 pid=103523 qemuPid=103538 sandbox=54632572e15297178d78997f88f8e7e4b66d1568c16a89e038e096aca0298c4a source=virtcontainers/hypervisor subsystem=qemu


sudo systemctl restart snap.microk8s.daemon-containerd.service

