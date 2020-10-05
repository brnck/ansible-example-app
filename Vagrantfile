Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.ssh.shell = "bash"
  config.vm.synced_folder '.', '/vagrant', disabled: true

  if ENV['DISTRIBUTION'] == 'buster'
    config.vm.box = "debian/buster64" # Debian 10
  elsif ENV['DISTRIBUTION'] == 'stretch'
    config.vm.box = "debian/stretch64" # Debian 9
  end


  config.vm.provider :virtualbox do |v|
    v.memory = 1024
    v.cpus = 1
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.define "gateway", primary: true do |gateway|
    gateway.vm.hostname = "gateway"
    gateway.vm.network "private_network", ip: "10.240.0.11"
  end

  [1,2,3].each do |i|
    config.vm.define "example-app-#{i}" do |app|
      app.vm.hostname = "example-app-#{i}"
      app.vm.network "private_network", ip: "10.240.0.2#{i}"
    end
  end

  config.vm.provision "base", type: "shell" do |shell|
    shell.inline = "sudo apt-get update && sudo apt-get install -y python"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "tests/playbook.yml"
    ansible.limit = "all"
    ansible.compatibility_mode = "2.0"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.compatibility_mode = "2.0"
    ansible.groups = {
      "gw-group" => ["gateway"],
      "example-app-web-group" => ["example-app-[1:3]"],
      "python-group" => ["example-app-[1:3]"]
    }
  end
end
