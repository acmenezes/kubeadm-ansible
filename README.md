# Kubernetes Advanced Networking/Storage Provisioning with Kubeadm
---
This project aims to deploy special networking and/or storage projects to be used with kubernetes such as VPP, DPDK, SR-iOV, kernel and OS custom performance tunning as well as allow the most comprehensive list of CNI based network plugins to be installed with kubernetes. Each network plugin or system feature will be deployed as an ansible role allowing users to create flexible playbooks to install the way they want only setting main variables and the roles needed to have a working solution.

Another goal is to have a single platform to launch kubernetes customized environments in order to compare network and storage solutions as well as prepare demos, workshops and  meetup presentations.

As networking, storage can be also customized and should be added in ansible's role set as well to allow customized persistent storage with performance comparison opening the doors for future storage considerations in data science projects working with containerized big data tools such as spark, hadoop, kafka and similar workloads.

### Target audience 
Container solutions archictects willing to better understand the ins and outs of custom, optimized networking and storage to meet their client's needs.

### Prerequisites
This project can be deployed int two flavors: Terraform or Vagrant. If you're using vagrant you need virtualbox or kvm/libvirt (plus the vagrant itself and the vagrant-libvirt plugin if you have libvirt) working in your mahchine. If you're using terraform read the particular cloud provider instructions once they are ready in the docs folder.

Other than that you'll need up to date python and ansible installed on the testing machine.
