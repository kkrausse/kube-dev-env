# adapted from:
# https://medium.com/@lizrice/kubernetes-in-vagrant-with-kubeadm-21979ded6c63


Vagrant.configure("2") do |config|

  # number of nodes is N+1
  N = 2
  # do the same thing for each node
  (0..N).each do |node_id|
    config.vm.define "node#{node_id}" do |node|

      # base image ubuntu 18.04
      node.vm.box = "ubuntu/bionic64"
      # set up network to connect tap interface
      node.vm.network "public_network", ip: "192.168.99.#{20+node_id}", bridge: "tap0"
      # have docker installed on the machine
      node.vm.provision "docker"
      node.vm.hostname = "node#{node_id}"
      
      node.vm.provision :ansible do |ansible|
        anisble.host_vars = {
          "node#{node_id}" =>
        }
      end
      # Only execute once the Ansible provisioner,
      # when all the machines are up and ready.
      # this allows ansible to provision multiple nodes in parallel
      if node_id == N
        node.vm.provision :ansible do |ansible|
          # Disable default limit to connect to all the machines
          ansible.limit = "all"
          ansible.playbook = "playbook.yml"
          ansible.groups = {
            "masters" => ["node0"],
            "workers" => ["node[1:#{N}]"],
            "all:vars" => {
              "ansible_python_interpreter" => "/usr/bin/python3"
            }
          }

        end
      end
    end # config.vm...
  end # (0..N).each
end # configure
