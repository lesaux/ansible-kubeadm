# ansible-kubeadm
ansible scripts to deploy baremetal openstack clusters.

Currently either single master or ha-cluster of three masters is supported (You need either 1 or 3 nodes defined in the [masters] section of the hosts file.

When setup up a ha-cluster a prerequisite is to a have a load balancer fronting all three master nodes on port 6443, along with a dns record to resolve it.

The docker-ce module was taken on Galaxy.

## Addons 

The `addons.yml` tasks complete the installation by adding a few features to our fresh kubernetes cluster.

### K8S-Dashboard

It is made available on a public ip managed by metallb (see the metallb* variables in group_vars).

### Network CNI:
Currently either calico or weave are supported. My plan was to only use calico, but I have had trouble with it on VMware environemnts, which is why I now included weave.

### Load-Balancing:

For Load-Balancing, Metallb is included.

### Persistent-Volumes support

I have been testing Rook and the Freenas NFS provider.
Rook ceph is a little slow in my environment.

## Monitoring

### Elasticsearch

The `monitoring.yml` tasks deploys an ES cluster (pires) and all of the beats components. Kibana is installed and dashboards are provisioned.

### *-beats

Filebeat, packetbeat, metricbeat and auditbeat are installed.

## Adding new worker nodes to an existing cluster

The add_nodes.yml task refreshes a token for an existing master, and contructs a kubeadm join command which is then run on the new node that we want to add to our cluster.

This requires a new `hosts` files with one existing master, and only new worker nodes. see hosts_add_nodes file for an example.


## Upgrades

This is just an attempt. I went through an ha-cluster upgrade from v1.11.0 to v1.11.2. Chances of this failing are so high that's it probably risky to even try it. Let's hope for kubeadm to bring to use an easier upgrade process.
