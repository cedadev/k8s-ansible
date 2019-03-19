# k8s-ansible

This project aims to provide a simple [Ansible](https://www.ansible.com/) playbook to
provision a [Kubernetes](https://kubernetes.io) cluster. It uses the
[kubeadm tool](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
to set up a secure cluster.

The [Helm package manager](https://helm.sh/) is deployed into the cluster.

The [Nginx ingress controller](https://github.com/kubernetes/ingress/tree/master/controllers/nginx)
is also deployed to provide [Ingress resources](https://kubernetes.io/docs/concepts/services-networking/ingress/).

The playbook also aims to provide hooks for the integration of cloud providers. Currently, only
hooks for [OpenStack](https://www.openstack.org/) are implemented. The OpenStack integration
currently provides the following:

  * Kubernetes pulls machine info from Nova metadata.
  * A default `StorageClass` is installed, called `standard`, that provisions Cinder volumes on demand.
  * Kubernetes is configured to call out to Keystone to authenticate tokens using
    [k8s-keystone-auth](https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/using-keystone-webhook-authenticator-and-authorizer.md).
    A `ClusterRoleBinding` is installed that grants the `cluster-admin` role to all members of the OpenStack project in which the cluster is deployed.
    The `k8s-keystone-auth` project also provides a client that integrates with `kubectl` to provide OpenStack authentication.

# Deployment

    ansible-playbook cluster_openstack.yml -i inventory -e @config.yml

Where `config.yml` reads:

    ---
    cluster_name: k8s
    cluster_version: 1.13
    cluster_state: present
    cluster_network: caastest-U-internal
    cluster_keypair: brtknr-33236e18e6285585a7f58f1006f88349
    cluster_num_workers: 4
    cluster_gw_fip_ip: 192.171.139.249

    openstack_trustee_id: f8d4001b4e33ecdb24a11f3de1a9102275b3b39e7f1b3bad4caf33775d3f8c09
    openstack_trustee_password: 'secretnomore'
    ...
