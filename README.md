# MicroK8s
Installation of MicroK8s:

```bash
sudo snap install microk8s --classic --channel=1.33/stable
echo "alias k='microk8s kubectl'" >> ~/.bashrc
source ~/.bashrc
```

You may want to add `microk8s` to the suddoer group

```bash
sudo usermod -a -G microk8s $USER
mkdir -p ~/.kube
chmod 0700 ~/.kube
```

You will also need to logout for the change to take place.

# Operator
Operator manages the lifecycle of TDX workloads, by relaying on Katacontainer

**NOTE:** `roseblanche-1` correspond to the ssh config of the TDX vm, it should be adapted.

```bash
git clone https://github.com/cbarbieru/operator.git
cd ./operator
git checkout microk8s_patch

cd ${PWD}/config/release/
k apply -k .

k label node rosablanche-1 node.kubernetes.io/worker="" --overwrite

cd ${PWD}/config/samples/ccruntime/default
k apply -k .
```

## For operator pre-install errors
```bash 
sudo vim /var/snap/microk8s/current/args/config.toml.d/nydus-snapshotter.toml
```

<pre>
[proxy_plugins]
  [proxy_plugins.nydus]
	type = "snapshot"
	address = "/run/containerd-nydus/containerd-nydus-grpc.sock"
</pre>


```bash
sudo systemctl start nydus-snapshotter.service```

sudo vim /var/snap/microk8s/current/args/containerd-template.toml
```


<pre>
imports = ["/var/snap/microk8s/current/args/config.toml.d/*.toml"]
</pre>

## Configure custom QEMU
```bash
sudo vim /opt/kata/share/defaults/kata-containers/configuration-qemu-tdx.toml
```

<pre>
[hypervisor.qemu]
path = "/opt/kata/bin/qemu-system-x86_64-tdx-experimental"
# confidential_guest = true
</pre>
Latter is temp to avoid below
<pre>
time="2025-07-23T12:24:05.013714495Z" level=error msg="qemu-system-x86_64-tdx-experimental: vm-type TDX not supported by KVM" name=containerd-shim-v2 pid=103523 qemuPid=103538 sandbox=54632572e15297178d78997f88f8e7e4b66d1568c16a89e038e096aca0298c4a source=virtcontainers/hypervisor subsystem=qemu
</pre>

Restart containerd

```bash 
sudo systemctl restart snap.microk8s.daemon-containerd.service
```

# K8s Deployment
Allow to deploy the K8s setup. This can be done locally, simply by running this, or with `operator` and `katacontainer` on a TDX vm, by running the previous section.

```bash
git clone https://github.com/cbarbieru/builder-playground-opstack-k8s.git
cd ~/builder-playground-opstack-k8s
sudo mkdir -p /mnt/sceal/storage
sudo cp -a storage/. /mnt/sceal/storage
k apply -f ./resources/
```
