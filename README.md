## Running

taken some from:
https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-18-04

also a good one:
https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c

### Step 1: Install prerequisites

- Install Vagrant, VirtualBox, and Ansible
```
sudo apt-get install virtualbox
sudo apt-get install vagrant
sudo apt install ansible
```

### Step 2: Set up network

- Since Vagrant with VirtualBox failed to set up a private network, I will
  do this manually by running the following:
```
# add a tap interface to connect all nodes in the network and name it tap0
# all the vms will create a bridge to this interface and thus be connected
# the name "tap0" is important since its used in the vagrantfile
sudo ip tuntap add name tap0 mode tap
# set the ip that the host will use for the tap interface
sudo ip addr add 192.168.99.1 dev tap0
# put the tap interface up
sudo ip link set dev tap0 up
# configure your ip to route requests to this interface
sudo ip route add 192.168.99.0/24 dev tap0
```

### Step 3: Set up cluster

Open the Vagrantfile and set N to the number of worker nodes you want

Run `vagrant up` to set up the nodes and then `vagrant provision` to set up
the kubernetes cluster. This uses the ansible playbooks to configure all the
nodes.

### Step 4: Interracting with the cluster

To make things easier, you can set the node names in your `/etc/hosts` file.
I added the following lines:
```
192.168.99.20 node0.test.com node0
192.168.99.21 node1.test.com node1
```

You can run kubectl by sshing into the master (`ssh ubuntu@node0`) or you can
run it on the host machine by copying the kubernetes config file and putting it
in a readable location.

Run `scp ubuntu@node0:/home/ubuntu/.kube/config .` to copy the config to your
current directory and then run `export KUBECONFIG="$(pwd)/config"` so kubectl
knows to operate on this config.

Alternatively, you can just place the `config` file in your `~/.kube/` directory

Run `kubectl get nodes` to see that all your nodes are ready to roll

### Step 5: Add persistent volume claim

To do this I added the following line to `/etc/exports`:

`/home/kkrausse/projects/vag/shared *(rw,fsid=0,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)`

where `/home/kkrausse/projects/vag/shared` was the directory to mount for the nfs volume
then ran `sudo exportfs -a` to update the nfs server that was already running

Then did `kubectl apply -f pv.yml` to put that on there

#### Vocabulary

- when I say "host machine," I am talking about the machine that vagrant and
  and virtual box are installed on. This is opposed to "guest machine" which
  is one that is virtual.


# TODO

- change the workers' ips so they are in the kubernetes CIDR
  (10.244.0.0/16 in this case). This should solve the issue about pods on
  other nodes not being found without having to change the default kubelet args
  in `/etc/default/kubelet`
  - nvm this is wrong. Only the pod-cider from within the node be a sub-cidr
    of the cluster-wide one. doesnt matter with the node ip
