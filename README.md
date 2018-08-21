# ansible-kubeadm

ansible scripts to deploy baremetal openstack clusters.

Currently either single master or ha-cluster of three masters is supported (You need either 1 or 3 nodes defined in the [masters] section of the hosts file.

When setup up a ha-cluster a prerequisite is to a have a load balancer fronting all three master nodes on port 6443, along with a dns record to resolve it.

An example haproxy configuration to achieve this:

```
frontend k8s-api
  bind 192.168.0.10:6443
  bind 127.0.0.1:6443
  mode tcp
  option tcplog
  default_backend k8s-api

backend k8s-api
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-api-1 192.168.0.104:6443 check
  server k8s-api-2 192.168.0.105:6443 check
  server k8s-api-3 192.168.0.106:6443 check
```
Where 192.168.0.104, 192.168.0.105 and 192.168.0.106 are your master nodes.


The docker-ce module was taken on Galaxy.

## 1. Addons 

The `addons.yml` tasks complete the installation by adding a few features to our fresh kubernetes cluster.

This is done by applying new manifests to the cluster with kubectl. All manifests are ansible templates which are rendered in the `dynamically_rendered` folder before `kubectl apply` is run

### 1.a K8S-Dashboard

It is made available on a public ip managed by metallb (see the metallb* variables in group_vars).

### 1.b Network CNI:
Currently either calico or weave are supported. My plan was to only use calico, but I have had trouble with it on VMware environemnts, which is why I now included weave.

### 1.c Load-Balancing:

For Load-Balancing, Metallb is included.

### 1.d Persistent-Volumes support

I have been testing Rook and the Freenas NFS provider.
Rook ceph is a little slow in my environment.

## 2. Monitoring

### 2.1 Elasticsearch

The `monitoring.yml` tasks deploys an ES cluster (pires) and all of the beats components. Kibana is installed and dashboards are provisioned. The data nodes are stateful sets.

### 2.2 *-beats

Filebeat, packetbeat, metricbeat and auditbeat are installed.

### heapster

Is getting old, but still provides nice widgets to the k8s-dashboard.

### kube-state-metrics

beats relies on this service to obtain k8s metrics.

## 3. Adding new worker nodes to an existing cluster

The add_nodes.yml task refreshes a token for an existing master, and contructs a kubeadm join command which is then run on the new node that we want to add to our cluster.

This requires a new `hosts` files with one existing master, and only new worker nodes. see hosts_add_nodes file for an example.


## 4. Upgrades

This is just an attempt. I went through an ha-cluster upgrade from v1.11.0 to v1.11.2. Chances of this failing are so high that's it probably risky to even try it. Let's hope for kubeadm to bring to us an easier upgrade process.
