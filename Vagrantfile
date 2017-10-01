#BOX_IMAGE = "centos/atomic-host"
BOX_IMAGE = "ubuntu/xenial64"
NODE_COUNT = 2

Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 1
    vb.linked_clone = true
  end

  config.vm.define "rancherMaster", primary: true do |master|
    master.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
    end
    master.vm.box = BOX_IMAGE
    master.vm.hostname = "master.rancher.local"
    master.vm.network :private_network, ip: "192.168.34.10"
    master.vm.provision "shell", inline: "curl https://releases.rancher.com/install-docker/1.12.6.sh | sh"
    master.vm.provision "docker" do |d|
      d.run "rancher-master",
        image: "rancher/server:preview",
        restart: "unless-stopped",
        args: "-p 8080:8080"
    end
    master.vm.provision "shell", inline: <<-EOC
      while ! curl --output /dev/null --silent --head --fail http://localhost:8080; do sleep 10 && \
      echo -n "waiting rancher to come up..."; done;
      curl -X PUT -d 'value=192.168.34.10:8080' \
      http://localhost:8080/v2-beta/settings/api.host
    EOC
  end

  (1..NODE_COUNT).each do |i|
    config.vm.define "rancherNode#{i}" do |node|
      node.vm.box = BOX_IMAGE
      node.vm.hostname = "node#{i}.rancher.local"
      node.vm.network :private_network, ip: "192.168.34.#{i + 10}"
      node.vm.provision "shell", inline: <<-EOC
        curl https://releases.rancher.com/install-docker/1.12.6.sh | sh
        apt-get install -y python-minimal
        response=$(curl -sb -H "Accept: application/json" \
        "http://master.rancher.local:8080/v2-beta/clusters/1c1" | \
        python -m json.tool)
        curl http://stedolan.github.io/jq/download/linux64/jq -o /usr/local/bin/jq
        chmod +x /usr/local/bin/jq
        echo $response | /usr/local/bin/jq -r .registrationToken.hostCommand | sh
      EOC
    end
  end

end
