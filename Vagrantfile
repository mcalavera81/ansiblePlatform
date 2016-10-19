Vagrant.configure("2") do |config|

  #
  # Run Ansible from the Vagrant Host
  #
 config.vm.define "ansible" do |ansible_config|
      ansible_config.vm.box = "ubuntu/trusty64"
      ansible_config.vm.hostname = "ansible"
      ansible_config.vm.network :forwarded_port, guest: 8080, host: 8081 
      ansible_config.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
      end
      ansible_config.vm.provision "ansible" do |ansible|
	ansible.raw_ssh_args = ['-o ForwardAgent=yes']
        ansible.playbook = "playbook.yml"
      end	
  end
end
