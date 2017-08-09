# -*- mode: ruby -*-
# vi: set ft=ruby :

N_NODES = 2


Vagrant.configure(2) do |config|
  config.vm.box = "boxcutter/ubuntu1604"

  config.vm.provider :virtualbox do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end

  # Allow root login with same key as vagrant user
  config.vm.provision :shell, inline: <<-SHELL
echo "Copying SSH key to root..."
mkdir -p /root/.ssh
cp ~vagrant/.ssh/authorized_keys /root/.ssh
SHELL

  config.vm.define "kube-proxy" do |master|
    master.vm.hostname = "kube-proxy"
    master.vm.network "private_network", ip: "172.28.128.100"
  end

  (0..N_NODES-1).each do |n|
    node_name = "kube-node%d" % n
    config.vm.define node_name do |node|
      node.vm.hostname = node_name
      node.vm.network "private_network", ip: "172.28.128.%s" % (101 + n)

      if n == (N_NODES-1)
        # On the final node (i.e. when all the machines in the cluster have started)
        # we run the playbook
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "cluster.yml"
          # ansible.verbose = "vvvv"
          ansible.limit = "all"
          ansible.force_remote_user = false
          ansible.groups = {
            "proxy_hosts" => ["kube-proxy"],
            "kube_masters"  => ["kube-node0"],
            "kube_nodes" => (1..N_NODES-1).map { |n| "kube-node%d" % n },
            "kube_hosts:children" => ["kube_masters", "kube_nodes"],
            "vagrant_hosts:children" => ["proxy_hosts", "kube_hosts"]
          }
        end
      end
    end
  end
end
