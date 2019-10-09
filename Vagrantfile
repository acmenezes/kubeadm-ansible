Vagrant.configure("2") do |config|
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

	config.vm.provider "libvirt" do |libvirt|
		libvirt.memory = 4096
		libvirt.cpus = 2
    #libvirt.nic_model_type = "virtio"
    #libvirt.driver = "kvm"
    #libvirt.nested = true		
	end
	
	config.vm.provider "virtualbox" do |virtualbox|
		virtualbox.memory = 4096
		virtualbox.cpus = 2
	end
	
	config.vm.define "master" do |master|
			master.vm.hostname = "master"
			master.vm.box = "centos/7"
			#master.vm.network "private_network", type: "dhcp"
			master.vm.network "private_network", ip: "192.168.255.100"
			master.vm.provision "ansible" do |ansible|
				# ansible.playbook = "deploy_multus.yml"
				ansible.playbook = "deploy_master.yml"				
				# ansible.tags = "test_rke"
				ansible.groups = {
					"master_nodes" => ["master"]
				}
			end
	end

	N = 1
	(1..N).each do |node_number|
		config.vm.define "node#{node_number}" do |node|
				node.vm.hostname = "node#{node_number}"
				node.vm.box = "centos/7"
				node.vm.network "private_network", ip: "192.168.255.10#{node_number}"
				node.vm.provision "ansible" do |ansible|
					ansible.playbook = "deploy_node.yml"
					# ansible.tags = "test_rke"
					ansible.groups = {
						"master_nodes" => ["master"],
						"worker_nodes" => ["node1","node2","node3"]
					}
				end
		end
	end
end
