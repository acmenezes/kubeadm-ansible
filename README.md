# A Simple Multi-Node Kubernetes Lab Installer
---
Here you can find a very simple and quick recipe to spin up a local multi-node kubernetes lab environment.

### Requirements (and versions tested)

- Vagrant 2.2.19
- vagrant-libvirt plugin 0.7.0
- Ansible 2.9.10

### How to Use

The Vagrantfile has the most important configuration options:

#### Memory and CPU

You can change those values accorging to your hardware capacity although kubernetes itself will have its minimal requirements.

```
		libvirt.memory = 4096
		libvirt.cpus = 2
```

#### Provider

This script is supposed to work with KVM and libvirt. If you need a different one such as virtualbox you can set your own changing the section that starts with:
```
config.vm.provider "libvirt" do |libvirt|
```

#### Number of Worker Nodes

You can change the number of worker nodes by changing the variable N. Remember that total memory used is the configured memory above times N plus 1 which accounts for the master node. In the default configuration it will take up to 12GB of memory (2X4GB worker nodes + 1x4GB master node).
```
	N = 2
	(1..N).each do |node_number|
```
#### NOTE:
> :warning: This script disables SELinux and Firewalld to simplify development tasks.