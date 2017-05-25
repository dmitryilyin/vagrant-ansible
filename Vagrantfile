# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'erb'

machines = {
  'ansible-main' => '192.168.0.2',
  'ansible-1' => '192.168.0.3',
  'ansible-2' => '192.168.0.4',
  'ansible-3' => '192.168.0.5',
}

inventory_template = <<-eof
[all]
<%- machines.each do |host, ip| -%>
<%= host %> ansible_host='<%= ip %>'
<%- end -%>

[all:vars]
ansible_connection=ssh
ansible_user=root
ansible_port=22
eof

hosts_template = <<-eof
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
<%- machines.each do |host, ip| -%>
<%= ip %> <%= host %>
<%- end -%>
eof

inventory_file = ERB.new(inventory_template, nil, '-').result(binding)
hosts_file = ERB.new(hosts_template, nil, '-').result(binding)

inventory_script = <<-eof
mkdir -p /etc/ansible
cat << EOF > /etc/ansible/hosts
#{inventory_file}
EOF
eof

hosts_script = <<-eof
cat << EOF > /etc/hosts
#{hosts_file}
EOF
eof

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  machines.each do |host, ip|
	config.vm.define host do |vm|
		vm.vm.hostname = host
		
		vm.vm.network "private_network", ip: ip

		vm.vm.synced_folder ".", "/vagrant", type: "virtualbox"
		vm.vm.synced_folder 'provisioning/roles', '/etc/ansible/roles', type: "virtualbox"

		vm.vm.provision "ansible_local" do |ansible|
		  ansible.install_mode = 'default'
		  ansible.verbose = true
		  ansible.install = true
		  ansible.groups = {
			"carbon-servers" => ["carbon"],
		  }
		  ansible.playbook = "provisioning/site.yml"
        end
		
		vm.vm.provision "shell", inline: inventory_script
		vm.vm.provision "shell", inline: hosts_script

	end
  end

end
