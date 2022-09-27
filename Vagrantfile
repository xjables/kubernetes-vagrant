require "yaml"

vars = YAML::load_file("./config.yml")

BASE_IMAGE            = vars['BASE_IMAGE']
CONTROL_COUNT         = vars['CONTROL_COUNT']
INITIAL_CONTROLLER_IP = vars['INITIAL_CONTROLLER_IP']
NODE_COUNT            = vars['NODE_COUNT']
INITIAL_NODE_IP       = vars['INITIAL_NODE_IP']

ansible_groups = {
  "nodes" => [ "k8s-node-[1:#{NODE_COUNT}]" ],
  "controllers" => [ "k8s-controller-[1:#{CONTROL_COUNT}]" ],
}

def increment_ip(ip, inc_value)
    ip_arr = ip.split(".")

    # Increment last IP value and rejoin
    new_ip_arr = ip_arr[0..2].append((ip_arr[-1].to_i + inc_value).to_s)
    return new_ip_arr.join(".")
end


Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end
      
	(1..CONTROL_COUNT).each do |i|
	    CONTROLLER_IP   = increment_ip INITIAL_CONTROLLER_IP, i - 1
        config.vm.define "k8s-controller-#{i}" do |controller|
            controller.vm.box = BASE_IMAGE
            controller.vm.network "private_network", ip: CONTROLLER_IP
            controller.vm.hostname = "k8s-controller-#{i}"
            controller.vm.provision "ansible" do |ansible|
              ansible.groups = ansible_groups
              ansible.limit = "controllers"
              ansible.playbook = "ansible/common-playbook.yml"
              ansible.extra_vars = {
                node_ip: CONTROLLER_IP,
              }
            end
			controller.vm.provision "ansible" do |ansible|
              ansible.groups = ansible_groups
              ansible.limit = "controllers"
              ansible.playbook = "ansible/controller-playbook.yml"
              ansible.extra_vars = {
                node_ip: CONTROLLER_IP,
				node_name: "k8s-controller-#{i}"
              }
            end
        end
    end

    (1..NODE_COUNT).each do |i|
		    NODE_IP       = increment_ip INITIAL_NODE_IP, i - 1
        config.vm.define "k8s-node-#{i}" do |node|
          node.vm.box = BASE_IMAGE
            node.vm.network "private_network", ip: NODE_IP
            node.vm.hostname = "k8s-node-#{i}"

            if i == NODE_COUNT
              node.vm.provision "ansible" do |ansible|
                ansible.groups = ansible_groups
                ansible.limit = "nodes"
                ansible.playbook = "ansible/common-playbook.yml"
                ansible.extra_vars = {
                  node_ip: NODE_IP,
                }
              end
              node.vm.provision "ansible" do |ansible|
                ansible.groups = ansible_groups
                ansible.limit = "nodes"
                ansible.playbook = "ansible/node-playbook.yml"
                ansible.extra_vars = {
                  node_ip: NODE_IP,
					        node_name: "k8s-node-#{i}"
                }
              end
            end
        end
    end
end