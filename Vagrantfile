Vagrant.configure("2") do |config|

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

	config.vm.provider "libvirt" do |libvirt|
		libvirt.memory = 4096
		libvirt.cpus = 2

	# If doing nested virtualization uncomment the
	# options below
    # libvirt.nic_model_type = "virtio"
    # libvirt.driver = "kvm"
    # libvirt.nested = true		
	end
	
	config.vm.define "master" do |master|
			master.vm.hostname = "master"
			master.vm.box = "generic/fedora33"
			config.vm.network "public_network", :dev => "virbr0", :mode => 'bridge', :type => "bridge"
			master.vm.provision "ansible" do |ansible|
				ansible.playbook = "ansible/k8s-control-plane.yml"		
				ansible.groups = {
					"master_nodes" => ["master"]
				}
			end	
	end

	# If you need more worker nodes on your lab increase N
	# Just make sure you have enough memory to run extra nodes
	N = 2
	(1..N).each do |node_number|
		config.vm.define "node#{node_number}" do |node|
				node.vm.hostname = "node#{node_number}"
				node.vm.box = "generic/fedora33"
				config.vm.network "public_network", :dev => "virbr0", :mode => 'bridge', :type => "bridge"					
				node.vm.provision "ansible" do |ansible|
					ansible.playbook = "ansible/k8s-worker-nodes.yml"
					ansible.groups = {
						"master_nodes" => ["master"],
						"worker_nodes" => ["node1","node2"]
					}
				end
		end
	end
end
