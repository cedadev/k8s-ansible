# k8s-ansible

This project aims to provide an [Ansible](https://www.ansible.com/) playbook to
provision a Kubernetes cluster.

It uses the [kubeadm tool](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
to set up a secure cluster.

Storage is provided by [Ceph](http://ceph.com/) deployed using [rook](https://rook.io/),
with a default [StorageClass](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storageclasses)
for Kubernetes dynamically provisioned storage.

The [Nginx ingress controller](https://github.com/kubernetes/ingress/tree/master/controllers/nginx)
is also deployed to provide [Ingress resources](https://kubernetes.io/docs/concepts/services-networking/ingress/).


## Deploying the test cluster

The test cluster is deployed using [Vagrant](https://www.vagrantup.com/) and
[VirtualBox](https://www.virtualbox.org/), so make sure you have recent versions
of both installed.

By default the Kubernetes cluster consists of a master and 2 minions. The number
of minions can be increased by changing `N_NODES` at the top of the `Vagrantfile`.

To provision the test cluster, just run `vagrant up`.

To access `kubectl` on the master, just use:

```
$ vagrant ssh kube-master -- -l root
[root@kube-master ~]# kubectl ...
```

Alternatively, you can install `kubectl` on your host system, and use the config
file that was pulled from the `kube-master` during the Ansible playbook:

```
$ export KUBECONFIG=$HERE/.artifacts/vagrant/kubernetes-admin.conf
$ kubectl ...
```

This method is particularly useful for accessing the API server, and any dashboards
you choose to install, using
[kubectl proxy](https://kubernetes.io/docs/tasks/access-kubernetes-api/http-proxy-access-api/).
