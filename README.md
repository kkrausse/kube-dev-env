## Running

#### Step 1: install prereqs

- install vagrant, virtualbox, and ansible

#### Step 2: set up network

- since vagrant with virtual box failed to set up a private network I will
  do this manually by running the following:
```
# add a tap interface to connect all nodes in the network and name it tap0
# the name "tap0" is important since its used in the vagrantfile
sudo ip tuntap add name tap0 made tap
# set the ip that the host will use for the tap interface
sudo ip addr add 192.168.99.1 dev tap0
# TODO: I dont know if this command is actually necessary right now, but
# at some point, the tap0 interface needs to be up
sudo ip link set dev tap0 up
# configure your ip to route requests to this interface
sudo ip route add 192.168.99.0/24 dev tap0
```

#### vocabulary

- when I say "host machine," I am talking about the machine that vagrant and
  and virtual box are installed on. This is opposed to "guest machine" which
  is one that is virtual.
