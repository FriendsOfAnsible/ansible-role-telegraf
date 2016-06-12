# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.define 'ubuntu' do |config|
    config.vm.box = "ubuntu/trusty64"

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "tests/ubuntutest.yml"
      ansible.raw_arguments  = [
         "--sudo",
         "-vvv"
      ]
    end
  end
  
  config.vm.define 'windows' do |config|
    config.vm.box = "opentable/win-2012r2-standard-amd64-nocm"
    config.vm.communicator = "winrm"    

    # Execute the Ansible script just in case the image is missing something
    config.vm.provision "shell", path: "ConfigureRemoteAnsible.ps1"

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "tests/wintest.yml"
      
      # vagrant automatic inventory seems to still be using 
      # the old Ansible 1.x arguments which makes Ansible fail
      ansible.inventory_path = "tests/vagrant_inventory"
      ansible.raw_arguments  = [
         "-vvv"
      ]

    end

    # Allowing RDP with Microsoft Remote desktop and vagrant rdp
    config.vm.network "forwarded_port", guest: 3389, host: 13389
  end
    

end
