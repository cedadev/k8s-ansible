# -*- mode: ruby -*-
# vi: set ft=ruby :

N_WORKERS = 2

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"

  # Allow root login with same key as vagrant user
  config.vm.provision :shell, inline: <<-SHELL
    set -ex
    mkdir -p /root/.ssh
    cp ~vagrant/.ssh/authorized_keys /root/.ssh
  SHELL

  # Due to a Vagrant bug, eth1 stays down on boot on CentOS 7
  #   Here, we make sure it starts on each boot
  #   See https://github.com/mitchellh/vagrant/issues/8166
  config.vm.provision :shell, run: "always", inline: <<-SHELL
    set -ex
    ifup eth1
  SHELL

  # Make sure Kubernetes service traffic is routed over eth1
  # This is only necessary because eth1 is not the default interface
  # We can't just make eth1 the default interface because it won't route traffic to the internet
  config.vm.provision :shell, inline: <<-SHELL
    set -exo pipefail
    my_ip="$(ip addr show eth1 | grep "inet " | awk '{ print $2; }' | cut -d "/" -f 1)"
    echo "10.96.0.0/12 via ${my_ip} dev eth1" > /etc/sysconfig/network-scripts/route-eth1
    # For some reason, the first restart sometimes, but not always, refuses to pick up the new route
    # Running two restarts together seems to make this more reliable...
    systemctl restart network
    systemctl restart network
    ip route list
  SHELL

  # Give each host 2 CPUs and 2GB RAM
  config.vm.provider :virtualbox do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end

  config.vm.define "kube-master" do |master|
    master.vm.hostname = "kube-master"
    master.vm.network :private_network, ip: "172.28.128.100"
  end

  (1..N_WORKERS).each do |n|
    host_name = "kube-worker%d" % n
    config.vm.define host_name do |node|
      node.vm.hostname = host_name
      node.vm.network :private_network, ip: "172.28.128.%s" % (100 + n)

      if n == N_WORKERS
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "cluster_configure.yml"
          # ansible.verbose = "vvvv"
          ansible.limit = "all"
          ansible.force_remote_user = false
          ansible.groups = {
            "kube-masters"  => ["kube-master"],
            "kube-workers" => (1..N_WORKERS).map { |n| "kube-worker%d" % n },
          }
          ansible.extra_vars = {
            "cluster_name" => "vagrant",
            "cluster_interface" => "eth1",
          }
        end
      end
    end
  end
end
