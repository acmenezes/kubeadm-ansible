# Kubernetes Advanced Lab Configurations via Ansible and Kubeadm
---
The main goal for this project is to have a handy tool to quickly spin up a local kubernetes environment with custom configurations that are not possible with minikube. When at least more than two nodes are needed or to test high availability features when we have multiple nodes.

So different CNI plugins, storage plugins or special projects such as multus, kubevirt and others can be installed according to the variables input on the main playbook.

This way we have a single tool to launch customized kubernetes environments in order to compare network and storage solutions as well as prepare demos, workshops and meetup presentations.

This is accomplished using vagrant and libvirt for virtualization as well as ansible to install everything needed on the nodes. In order to use this project it's necessary to have a linux box running libvirt/kvm. 

### Target audience 

Container solutions archictects willing to better understand the ins and outs of custom, optimized networking and storage to meet their client's needs.

### Prerequisites

This project needs vagrant with kvm/libvirt working in a Linux box. If 
Other than that you'll need an up to date python and ansible installation on the testing machine.
