# -*- mode: ruby -*-
# vi: set ft=ruby :

N_MINIONS = 2

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"

  # Allow root login with same key as vagrant user
  config.vm.provision :shell, inline: <<-SHELL
    mkdir -p /root/.ssh
    cp ~vagrant/.ssh/authorized_keys /root/.ssh
  SHELL

  # Due to a Vagrant bug, eth1 stays down on boot on CentOS 7
  #   Here, we make sure it starts on each boot
  #   See https://github.com/mitchellh/vagrant/issues/8166
  config.vm.provision :shell, run: "always", inline: "ifup eth1"

  # Make sure Kubernetes service traffic is routed over eth1
  # This is only necessary because eth1 is not the default interface
  # We can't just make eth1 the default interface because it won't route traffic to the internet
  config.vm.provision :shell, inline: <<-SHELL
    my_ip="$(ip addr show eth1 | grep "inet " | awk '{ print $2; }' | cut -d "/" -f 1)"
    echo "10.96.0.0/12 via ${my_ip} dev eth1" > /etc/sysconfig/network-scripts/route-eth1
    systemctl restart network
  SHELL

  # Give each host 2 CPUs and 2GB RAM
  config.vm.provider :virtualbox do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end

  config.vm.define "kube-master" do |master|
    master.vm.hostname = "kube-master"
    master.vm.network :private_network, ip: "172.28.128.100"
  end

  (1..N_MINIONS).each do |n|
    host_name = "kube-minion%d" % n
    config.vm.define host_name do |node|
      node.vm.hostname = host_name
      node.vm.network :private_network, ip: "172.28.128.%s" % (100 + n)

      # Add an additional disk to each minion to trigger it as a storage node
      # This disk will be used to store Ceph volumes
      disk_path = './.vagrant/disks/%s.vdi' % host_name
      node.vm.provider :virtualbox do |vb|
        unless File.exists?(disk_path)
          vb.customize [
            'createhd',
            '--filename', disk_path,
            '--size', 50 * 1024
          ]
        end
        vb.customize [
          'storageattach',
          :id,
          '--storagectl', 'IDE',
          '--port', 1,
          '--device', 0,
          '--type', 'hdd',
          '--medium', disk_path
        ]
      end

      if n == N_MINIONS
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "cluster.yml"
          # ansible.verbose = "vvvv"
          ansible.limit = "all"
          ansible.force_remote_user = true
          ansible.become = true
          ansible.groups = {
            "kube_masters"  => ["kube-master"],
            "kube_minions" => (1..N_MINIONS).map { |n| "kube-minion%d" % n },
            "kube_hosts:children" => ["kube_masters", "kube_minions"],
          }
          ansible.extra_vars = {
            "cluster_interface" => "eth1",
            "userspace_proxy" => true,
            "rook_version" => "v0.8.0-4.gfb2c208",
          }
        end
      end
    end
  end
end
