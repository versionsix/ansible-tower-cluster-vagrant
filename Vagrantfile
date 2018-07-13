# install `vagrant plugin install vagrant-hostsupdater`
Vagrant.configure(2) do |config|
  # keep insecure key to make manual ansible
  # deployment to hosts possible. Handy for debugging, remove in unsecure env.
  config.ssh.insert_key = false
  # centos7 image is build by running following
  # (it speeds up the provisioning afterwards)
  # https://git.io/fbjxg
  # replace with generic/centos7 or centos/7 if you want to run with
  # vanilla vagrant cloud image (takes longer)
  # using virtualbox because vagrant-libvirt doesn't support linked_clone
  config.vm.box = "centos7"
  towers = Array.new
  ('a'..'c').each do |n|
    config.vm.define "tower-#{n}" do |subconfig|
      towers.push("tower-#{n}.example.com")
      subconfig.vm.hostname = "tower-#{n}.example.com"
      subconfig.vbguest.auto_update = false
      subconfig.vm.network :private_network, ip: "192.168.199.10#{n.ord-96}"
      subconfig.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
        vb.linked_clone = true
      end
      subconfig.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end
  databases = Array.new
  config.vm.define "db-1" do |subconfig|
    subconfig.vm.hostname = "database.example.com"
    databases.push("database.example.com")
    subconfig.vbguest.auto_update = false
    subconfig.vm.network :private_network, ip: "192.168.199.100"
    subconfig.vm.provider :virtualbox do |vb|
      vb.memory = 2048
      vb.cpus = 2
      vb.linked_clone = true
    end
    subconfig.vm.synced_folder ".", "/vagrant", disabled: true
    # Ansible provisioning: this is in the database part
    # to ensure the database is provisioned before the towers
    # and to make sure the provisioner is only ran once
    subconfig.vm.provision :ansible do |ansible|
      ansible.limit = "all"
      ansible.groups = {
          "tower" => ["tower-a","tower-b", "tower-c"],
          "databases"  => ["db-1"]
      }
      ansible.playbook = "deploy_tower_cluster.yml"
    end
  end
end
