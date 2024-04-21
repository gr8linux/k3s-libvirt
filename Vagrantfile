require 'yaml'

# Load inventory from YAML file
inventory = YAML.load_file(File.join(__dir__, 'inventory.yml'))

# Configuration variables
vm_box = "generic/ubuntu2204"
default_interface = "virbr0"  # Bridge interface

Vagrant.configure("2") do |config|
  inventory['all']['children'].each do |group, properties|
    properties['hosts'].each do |host, host_vars|
      config.vm.define host do |node|
        node.vm.box = vm_box
        node.vm.hostname = host

        # Setup public network; use a bridge with the default interface
        # If host_vars['ansible_host'] is undefined or nil, it will use DHCP automatically
        if host_vars.key?('ansible_host') && !host_vars['ansible_host'].nil?
          node.vm.network "private_network", bridge: default_interface, ip: host_vars['ansible_host']
        else
          node.vm.network "private_network", bridge: default_interface  # DHCP
        end

        # Configure libvirt provider specifics
        node.vm.provider "libvirt" do |v|
          v.memory = host_vars['memory']
          v.cpus = host_vars['cpu']
          # Additional disks can be added here if uncommented
          # v.storage :file, :size => '40G', :type => 'qcow2'
        end
      end
    end
  end

  # Timezone plugin configuration
  if Vagrant.has_plugin?("vagrant-timezone")
    config.timezone.value = :host
  end

  # SSH key provisioning
  config.vm.provision "file", source: "#{ENV['HOME']}/.ssh/id_new.pub", destination: "~/.ssh/me.pub"

  # Append public key to authorized_keys on the guest machine
  config.vm.provision "shell", inline: <<-SHELL
    cat /home/vagrant/.ssh/me.pub >> /home/vagrant/.ssh/authorized_keys
  SHELL
end
