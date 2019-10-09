## Multus Demo

The purpose of this lab is to showcase the multi-homed pod with multus. Multus was created in order to facilitate the migration of NFV workloads from traditional VMs to containers.


1) First we need to clone the Mutus github repository and move into the multus-cni plugin:

```
git clone https://github.com/intel/multus-cni.git && cd multus-cni
```

2) Then we may use the daemonset provided in the project to easily create the kubernetes objects:

```
cat ./images/{multus-daemonset.yml,flannel-daemonset.yml} | kubectl apply -f -
```
3) Now we need to wait for our nodes' status to be READY with the command below:

```
kubectl get nodes

--------------------------------------------------------------
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   58m   v1.13.3
node1    Ready    <none>   55m   v1.13.3

```
4) Multus uses the Custom Resource Definition CRD object to create new abstractions available on the kubernetes API. It means that we can create a new class of objects to be instantiated with an API RESTful call. In our case it's the Network Attachment Definition that we're going to use as a representation for the additional network interfaces.

Here is one for the macvlan protocol:

```
cat <<EOF | kubectl create -f -
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: macvlan-conf
spec:
  config: '{
      "cniVersion": "0.3.0",
      "type": "macvlan",
      "master": "eth0",
      "mode": "bridge",
      "ipam": {
        "type": "host-local",
        "subnet": "192.168.1.0/24",
        "rangeStart": "192.168.1.200",
        "rangeEnd": "192.168.1.216",
        "routes": [
          { "dst": "0.0.0.0/0" }
        ],
        "gateway": "192.168.1.1"
      }
    }'
EOF
```

5) Now we can check our definitions:

```
kubectl get network-attachment-definitions

--------------------------
NAME           AGE
macvlan-conf   21m

```

```
kubectl describe network-attachment-definitions macvlan-conf
```

6) Let's look into the details:

```
kubectl describe network-attachment-definitions macvlan-conf

-----------------------------------------------------------------------------------------------------
Name:         macvlan-conf
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  k8s.cni.cncf.io/v1
Kind:         NetworkAttachmentDefinition
Metadata:
  Creation Timestamp:  2019-05-14T01:12:54Z
  Generation:          1
  Resource Version:    4126
  Self Link:           /apis/k8s.cni.cncf.io/v1/namespaces/default/network-attachment-definitions/macvlan-conf
  UID:                 6712158d-75e5-11e9-86c2-52540093f677
Spec:
  Config:  { "cniVersion": "0.3.0", "type": "macvlan", "master": "eth0", "mode": "bridge", "ipam": { "type": "host-local", "subnet": "192.168.1.0/24", "rangeStart": "192.168.1.200", "rangeEnd": "192.168.1.216", "routes": [ { "dst": "0.0.0.0/0" } ], "gateway": "192.168.1.1" } }
Events:    <none>
```
7) Now that we have proper definitions we can create a pod calling additional interfaces from the kubernetes API according to our definitions using annonations on our pods. On our example below we create a pod with 2 interfaces. The primary is flannel and doesn't need any annotation and then we create a macvlan type interface:

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: samplepod
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-conf
spec:
  containers:
  - name: samplepod
    command: ["/bin/bash", "-c", "sleep 2000000000000"]
    image: dougbtv/centos-network
EOF
```
8) Let's take a look into the pod and list the interfaces:

```
kubectl exec -it samplepod -- /bin/bash
``` 

```
ip address

[root@samplepod /]# ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
3: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP 
    link/ether 0a:58:0a:f4:01:0c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.1.12/24 scope global eth0
       valid_lft forever preferred_lft forever
4: net1@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
    link/ether 16:06:95:5a:61:e1 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.200/24 scope global net1
       valid_lft forever preferred_lft forever
```

```
arp -a

----------------------------------------------------------------------------------------------------
10-244-1-3.kube-dns.kube-system.svc.cluster.local (10.244.1.3) at 0a:58:0a:f4:01:03 [ether] on eth0
10-244-1-2.kube-dns.kube-system.svc.cluster.local (10.244.1.2) at 0a:58:0a:f4:01:02 [ether] on eth0
gateway (10.244.1.1) at 0a:58:0a:f4:01:01 [ether] on eth0
? (10.244.1.11) at 0a:58:0a:f4:01:0b [ether] on eth0
? (192.168.1.201) at 92:22:52:74:1e:bd [ether] on net1
```

```
ip route

-----------------------------------------------------------------------
default via 10.244.1.1 dev eth0 
10.244.0.0/16 via 10.244.1.1 dev eth0 
10.244.1.0/24 dev eth0  proto kernel  scope link  src 10.244.1.12 
192.168.1.0/24 dev net1  proto kernel  scope link  src 192.168.1.200 
```

```
kubectl get pods

NAME                                  READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
samplepod                             1/1     Running   0          24m   10.244.1.12   node1   <none>           <none>
samplepod2                            1/1     Running   0          17m   10.244.1.13   node1   <none>           <none>
```


```
kubectl exec -it samplepod2 -- /bin/bash

[root@samplepod2 /]# ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
3: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP 
    link/ether 0a:58:0a:f4:01:0d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.1.13/24 scope global eth0
       valid_lft forever preferred_lft forever
4: net1@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
    link/ether 92:22:52:74:1e:bd brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.201/24 scope global net1
       valid_lft forever preferred_lft forever
```

```
[root@samplepod2 /]# ping 192.168.1.200
PING 192.168.1.200 (192.168.1.200) 56(84) bytes of data.
64 bytes from 192.168.1.200: icmp_seq=1 ttl=64 time=0.055 ms
64 bytes from 192.168.1.200: icmp_seq=2 ttl=64 time=0.090 ms
64 bytes from 192.168.1.200: icmp_seq=3 ttl=64 time=0.099 ms
64 bytes from 192.168.1.200: icmp_seq=4 ttl=64 time=0.092 ms
```