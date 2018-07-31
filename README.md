# k8s-ansible

This project aims to provide a simple [Ansible](https://www.ansible.com/) playbook to
provision a [Kubernetes](https://kubernetes.io) cluster.

It uses the [kubeadm tool](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
to set up a secure cluster.

The [Helm package manager](https://helm.sh/) is deployed into the cluster.

The [Nginx ingress controller](https://github.com/kubernetes/ingress/tree/master/controllers/nginx)
is also deployed to provide [Ingress resources](https://kubernetes.io/docs/concepts/services-networking/ingress/).


## Deploying the test cluster

The test cluster is deployed using [Vagrant](https://www.vagrantup.com/) and
[VirtualBox](https://www.virtualbox.org/), so make sure you have recent versions
of both installed.

Dynamic storage is provided by a [StorageClass](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storageclasses)
provided by [rook](https://rook.io/), a project to make deploying containerised [Ceph](http://ceph.com/) easy.

By default the Kubernetes cluster consists of a master and 2 workers. The number
of workers can be increased by changing `N_WORKERS` at the top of the `Vagrantfile`.

To provision the test cluster, just run `vagrant up`.

To access `kubectl` on the master, just use:

```
$ vagrant ssh kube-master -- -l root
[root@kube-master ~]# kubectl ...
```

Alternatively, you can install `kubectl` on your host system. To do this, we need to pull
the config file from the `kube-master`:

```
$ SSHCONFIG="$(mktemp)"
$ vagrant ssh-config kube-master > $SSHCONFIG
$ scp -F $SSHCONFIG root@kube-master:/etc/kubernetes/admin.conf /path/to/kubeconfig.conf
$ export KUBECONFIG=/path/to/kubeconfig.conf
$ kubectl ...
```

This method is particularly useful for accessing the API server and any installed dashboards
using [kubectl proxy](https://kubernetes.io/docs/tasks/access-kubernetes-api/http-proxy-access-api/).
