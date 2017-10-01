# rancher-vagrant
Vagrantfile for the automated deployment of Rancher 2.0 Technical Preview on VirtualBox.

Pre-requisites
--------------
* VirtualBox
* [Vagrant plug-in Hostmanager](https://github.com/devopsgroup-io/vagrant-hostmanager)

Installation
------------

    $ git clone https://github.com/pipoe2h/rancher-vagrant.git
    $ cd rancher-vagrant

Configuration
-------------
The Rancher platform you get with this Vagrantfile is:
  * Single Rancher master server (2x CPU/2GB memory)
  * Two Rancher node servers (1x CPU/1GB memory)
  * Vagrant box Ubuntu/Xenial64
  
Before you run `vagrant up` you should review the Vagrantfile settings to map your requirements.

### Vagrant box
```ruby
BOX_IMAGE = "ubuntu/xenial64" # Rancher recommended OS
```

### Number of Rancher node servers
```ruby
NODE_COUNT = 2  # Minimum one node. Maximum 244 nodes
```

### Master server hardware
```ruby
master.vm.provider :virtualbox do |vb|
  vb.memory = 2048  # Recommended 4GB
  vb.cpus = 2
end
```

### Node server hardware
```ruby
config.vm.provider "virtualbox" do |vb|
  vb.memory = 1024
  vb.cpus = 1
  vb.linked_clone = true
end
```

### Hostname and IP address
**master**
```ruby
master.vm.hostname = "master.rancher.local"
master.vm.network :private_network, ip: "192.168.34.10"
```
**node**
For the node servers provisioning a loop is used. You should only change the three first octets of the IP address `192.168.34.` and not the formula `#{i + 10}`.
```ruby
node.vm.hostname = "node#{i}.rancher.local"
node.vm.network :private_network, ip: "192.168.34.#{i + 10}"
```

Usage
-----

    $ vagrant up
    
Depending on your hardware performance and Internet speed, a single Rancher node platform (master is always required) can take around 5-10 minutes to come up.

You can check the Rancher cluster status on ***http://master.rancher.local:8080/env/1a8/containers*** (System environment)
