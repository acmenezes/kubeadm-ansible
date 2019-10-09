
## Kubevirt Demo

This is a simple demo on how to run a VM alongside a container using the kubernetes API powered by kubevirt. This demo was executed on a test kubernetes cluster running weave as the CNI plugin.

  #### !!! Important requirement: If running on VMs you need to enable nested virtualization !!!

Deploying Kubevirt Operator:
```
export VERSION=v0.15.0
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
```

Deploying kubevirt:
(it may take up until 5 minutes to be in running state)
```
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```

Deploying a Test VM (check the vm.yaml in the end of this file - user/passwd are fedora/fedora):
```
kubectl apply -f vm.yaml
```
If we command kubectl get pods we should a special type of pod being scheduled:

```
NAME                                  READY   STATUS              RESTARTS   AGE
virt-launcher-testvmi-nocloud-cnfvc   0/2     ContainerCreating   0          38s
```
Now we may check our virtual machine.
```
kubectl get virtualmachineinstance
```
It should take sometime in scheduling state:
```
NAME              AGE   PHASE        IP    NODENAME
testvmi-nocloud   59s   Scheduling         

```
Once it's running we can see it's ip address
```
NAME              AGE   PHASE     IP          NODENAME
testvmi-nocloud   1m    Running   10.32.0.6   node2

```
Now let's deploy a test container:
```
kubectl run testconteiner --rm -i --tty --image nicolaka/netshoot -- /bin/bash

```
Take a look at the container's ip address and see that they are in the same network:
```
bash-4.4# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
29: eth0@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default 
    link/ether a6:75:5a:3c:a8:c5 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.32.0.7/12 brd 10.47.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
Now let's ping and see that they can communicate:
```
bash-4.4# ping 10.32.0.6
PING 10.32.0.6 (10.32.0.6) 56(84) bytes of data.
64 bytes from 10.32.0.6: icmp_seq=1 ttl=64 time=1.46 ms
64 bytes from 10.32.0.6: icmp_seq=2 ttl=64 time=0.878 ms
64 bytes from 10.32.0.6: icmp_seq=3 ttl=64 time=0.766 ms
64 bytes from 10.32.0.6: icmp_seq=4 ttl=64 time=0.817 ms
64 bytes from 10.32.0.6: icmp_seq=5 ttl=64 time=0.227 ms

```

### vm.yaml

```
apiVersion: kubevirt.io/v1alpha3
kind: VirtualMachineInstance
metadata:
  name: testvmi-nocloud
spec:
  terminationGracePeriodSeconds: 30
  domain:
    resources:
      requests:
        memory: 1024M
    devices:
      disks:
      - name: containerdisk
        disk:
          bus: virtio
      - name: emptydisk
        disk:
          bus: virtio
      - disk:
          bus: virtio
        name: cloudinitdisk
  volumes:
  - name: containerdisk
    containerDisk:
      image: kubevirt/fedora-cloud-container-disk-demo:latest
  - name: emptydisk
    emptyDisk:
      capacity: "2Gi"
  - name: cloudinitdisk
    cloudInitNoCloud:
      userData: |-
        #cloud-config
        password: fedora
        chpasswd: { expire: False }
```


