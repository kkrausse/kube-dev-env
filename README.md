## Running

taken some from:
https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-18-04

also a good one:
https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c

#### Step 1: install prereqs

- install vagrant, virtualbox, and ansible

#### Step 2: set up network

- since vagrant with virtual box failed to set up a private network I will
  do this manually by running the following:
```
# add a tap interface to connect all nodes in the network and name it tap0
# all the vms will create a bridge to this interface and thus be connected
# the name "tap0" is important since its used in the vagrantfile
sudo ip tuntap add name tap0 made tap
# set the ip that the host will use for the tap interface
sudo ip addr add 192.168.99.1 dev tap0
# put the tap interface up
sudo ip link set dev tap0 up
# configure your ip to route requests to this interface
sudo ip route add 192.168.99.0/24 dev tap0
```

### Step 3: set up cluster

Open the Vagrantfile and set N to the number of worker nodes you want

Run `vagrant up` to set up the nodes and then `vagrant provision` to set up
the kubernetes cluster. This uses the ansible playbooks to configure all the
nodes.

### Step 4: interracting with the cluster

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

### Step 5: add default storage class

This will make it so persistant volume claims are automatically satisfied by
[dynamically provisioned](
https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) persistent
volumes.

In this case, I am running an nfs server on the host and using the [nfs client](
https://github.com/helm/charts/tree/master/stable/nfs-client-provisioner)
external provisioner.

##### run nfs on the host os

Run `sudo apt install nfs-kernel-server` to run the nfs server on the host and
add the following line to `/etc/exports`:
```
/home/kkrausse/projects/vag/shared *(rw,fsid=0,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)
```
where `/home/kkrausse/projects/vag/shared` was the directory to mount for the nfs volume
then ran `sudo exportfs -a` to update the nfs server that was already running

##### Setting up the nfs client external provisioner

While in the root directory of this repository, run
```
helm install --name nfs-client-release stable/nfs-client-provisioner -f nfs-client-vals.yml 
```
After this you should be good to run the jupyterhub helm [deployment](
https://zero-to-jupyterhub.readthedocs.io/en/latest/index.html) on this cluster

For reference this helm chart is based on this [repo](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client).
Code in `nfs-client/cmd/nfs-client-provisioner` implements the kubernetes volume provisioner [interface](
https://github.com/kubernetes-sigs/sig-storage-lib-external-provisioner/blob/master/controller/volume.go)
and the helm chart has a deployment that makes a pod that runs this one file
and creates a storage class that uses the provisioner.


delete the nfs client provisioner with:
```
helm delete nfs-client-release --purge
```

### vocabulary

- when I say "host machine," I am talking about the machine that vagrant and
  and virtual box are installed on. This is opposed to "guest machine" which
  is one that is virtual.
