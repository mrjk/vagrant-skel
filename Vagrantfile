# -*- mode: ruby -*-
# vim: set ft=ruby  tabstop=2 softtabstop=0 expandtab shiftwidth=2 smarttab :
# vi: set ft=ruby  tabstop=2 softtabstop=0 expandtab shiftwidth=2 smarttab :

# Date: 2015/12/16
# Version 0.5
# License: MIT
# Author: mrjk[dot]78[at]gmail[dot]com



########################################
# Simplified vagrant architecture spwaner
########################################

# Please edit the "arch.yml" file to build your own architecture.
# Do not edit what's bellow unless you know what you do (and Ruby)
Vagrant.require_version ">= 1.7.0"
VAGRANTFILE_API_VERSION = "2"
require 'yaml'


########################################
# Display banner
########################################

puts "===================================================="
puts "==            Welcome on easy-vagrant             =="
puts "===================================================="
puts ""


########################################
# Default configuration
########################################

conf_default = {
  'settings' => {
    'defaults' => {
      'flavor' => 'tiny',
      'box' => 'centos7',
			'provider' => 'libvirt',
			'prefix' => 'prefix-',
			'sufix' => '-sufix',
    },
    'flavors' => {
      'micro' => {
        'cpu' => '1',
        'memory' => '128',
        'disk' => '10'
      },
      'tiny' => {
        'cpu' => '2',
        'memory' => '512',
        'disk' => '10'
      },
      'small' => {
        'cpu' => '2',
        'memory' => '1024',
        'disk' => '10'
      },
      'medium' => {
        'cpu' => '2',
        'memory' => '2024',
        'disk' => '10'
      },
      'large' => {
        'cpu' => '3',
        'memory' => '4096',
        'disk' => '10'
      },
      'huge' => {
        'cpu' => '4',
        'memory' => '8192',
        'disk' => '10'
      }
    },
    'boxes' => {
    	'debian8' => 'wholebits/debian8-64',
    	'debian7' => 'wholebits/debian7-64',
    	'ubuntu1604' => 'wholebits/ubuntu16.10-64',
    	'ubuntu1404' => 'wholebits/ubuntu14.04-64',
    	'centos7' => 'wholebits/centos7',
    	'centos6' => 'wholebits/centos5-64',
    	'arch' => 'wholebits/arch-64',
    },
    'providers' => {
      'libvirt' => {
				'driver' => 'nil',
				'host' => 'nil',
				'connect_via_ssh' => 'nil',
				'username' => 'nil',
				'password' => 'nil',
				'id_ssh_key_file' => '~/.ssh/id_rsa',
				'socket' => 'nil',
				'uri' => 'nil'
      },
      'virtualbox' => {
				'instances' => {
					'linked_clones' => 'true',
					'execution_cap' => '50',
					'page_fusion' => 'on',
					'cpuhotplug' => 'on',
					'pae' => 'on',
					'largepages' => 'on',
					'guestmemoryballoon' => '128',
        'boxes' => {
					'debian8' => 'minimal/jessie64',
					'debian7' => 'minimal/wheezy64',
					'ubuntu1604' => 'minimal/xenial64',
					'ubuntu1404' => 'minimal/trusty64',
					'centos7' => 'minimal/centos7',
					'centos6' => 'minimal/centos6',
        }
      	},
      },
      'docker' => {
				'force_host_vm' => 'false',
				'pull' => 'false',
				'remains_running' => 'true',
				'stop_timeout' => '30',
        'boxes' => {
					'scratch' => 'scratch',
					'alpine' => 'alpine:latest',
					'debian8' => 'debian:8',
					'debian7' => 'debian:7',
					'ubuntu1604' => 'ubuntu:xenial',
					'ubuntu1404' => 'ubuntu:trusty',
					'ubuntu1204' => 'ubuntu:precise',
					'centos7' => 'centos:7',
					'centos6' => 'centos:6',
					'centos5' => 'centos:5',
					'arch' => 'archlinuxjp/archlinux'
        }
      }
		}
  }
}


########################################
# Define functions
########################################

def value_is_defined(object, key)
  #puts "DEBUUUG" , object.class

  if object.class ==  Hash and object.key?(key)
    value = object[key].to_s
    if value.casecmp("nil") != 0
      return true
    end
  end
  return false
end

def merge_recursively(a, b)
    a.merge(b) {|key, a_item, b_item| 
      if a_item.class == Hash and b_item.class == Hash
        merge_recursively(a_item, b_item) 
			#elsif b_item.class == String and b_item.casecmp("nil") != 0
      #  key = b_item
			else
				key = b_item
      end
    }
end


########################################
# Load and merge config settings
########################################


# Load default configuration
conf_merged = conf_default

# Load mainteneur configuration
if File.file?('conf/vagrant.yml')
	conf_merged = merge_recursively(conf_merged, YAML.load_file('conf/vagrant.yml') )
end

# Load user configuration
if File.file?('local.yml')
	conf_merged = merge_recursively(conf_merged, YAML.load_file('local.yml') )
end

# Dump configuration
#print YAML::dump(conf_merged)


########################################
# Resolve configuration links
########################################

conf_final = {}
conf_final['providers'] = conf_merged['settings']['providers']


# Settings: Boxes
# =====================
conf_merged['settings']['providers'].each do |key, value|

  # Get common boxes
  global_boxes = conf_merged['settings']['boxes']

  # Get local provider box override
  if value_is_defined(value, 'boxes')
    provider_boxes = value['boxes']
  else
    provider_boxes = {}
  end

  # Update config
  conf_final['providers'][key]['boxes'] = merge_recursively(global_boxes, provider_boxes)

end


# Instances: flavors, boxes, number
# =====================
conf_final['instances'] = {}

conf_merged['instances'].each do |key, value|

  vm_config = {}

  # Get flavor name
  if value_is_defined(value, 'flavor')
    # Instance flavor is set
    vm_flavor = value['flavor']
  else
    # Fall back on default flavor
    vm_flavor = conf_merged['settings']['defaults']['flavor']
  end

  # Define disk
  if( value_is_defined(value, 'cpu') )
    vm_config['cpu'] = value['cpu']
  else
    vm_config['cpu'] = conf_merged['settings']['flavors'][vm_flavor]['cpu']
  end

  # Define memory
  if( value_is_defined(value, 'memory') )
    vm_config['memory'] = vm_config['cpu'] = value['memory']
  else
    vm_config['memory'] = conf_merged['settings']['flavors'][vm_flavor]['memory']
  end

  # Define disk
  if( value_is_defined(value, 'disk') )
    vm_config['disk'] = value['disk']
  else
    vm_config['disk'] = conf_merged['settings']['flavors'][vm_flavor]['disk'] 
  end


  # Define box
  if( value_is_defined(value, 'box') )
    vm_config['box'] = value['box']
  else
    vm_config['box'] = conf_merged['settings']['defaults']['box'] 
  end

  # Detect how many instances
  if value_is_defined(value, 'number') and value['number'] > 1

    # Generate the request number of instances
    for number in 1..value['number'] do
      # TODO: Do IP increment here
      conf_final['instances'][key + number.to_s] = vm_config.clone
    end

  elsif (
      value_is_defined(value, 'number') \
      and value['number'] != 0 \
    ) or not value_is_defined(value, 'number')

    # Write config
    conf_final['instances'][key] = vm_config
  end


end

# Print debug
#print YAML::dump(conf_final)

#abort



########################################
# Magic business
########################################

Vagrant.configure(2) do |config|

  conf_final['instances'].each do |key, value|

    # Display instance config
    # =====================
    puts "Define instance: " + key
    print YAML::dump(value)
    puts ""


    # Define instance
    # =====================
    config.vm.define key do |instance|


      # Generic Instance Definition
      # =====================
      instance.vm.hostname = key

      # Disable synced folders
      instance.vm.synced_folder ".", "/vagrant", :disabled => true


      # Provider: Libvirt
      # =====================
      instance.vm.provider "libvirt" do |l, o|

        # instanceure provider
        l.cpus = value["cpu"].to_i
        l.memory = value["memory"].to_i

        # Override
        o.vm.box = conf_final['providers']['libvirt']['boxes'][value["box"]]
      end
    

      # Provider: VirtualBox
      # =====================
      instance.vm.provider "virtualbox" do |v, o|

        # instanceure provider
        v.customize ["modifyvm", :id, "--cpus", value["cpu"]]
        v.customize ["modifyvm", :id, "--memory", value["memory"]]

        # Override
        o.vm.box = conf_final['providers']['virtualbox']['boxes'][value["box"]]
      end
    

      # Provider: Docker
      # =====================
      # Note: Todo

    end

  end


end















#################### OLD DEPRECATED

#       # I do not recommand you to use virtualbox, network settings are f***ing buggy/broken ...
#       config.vm.provider "libvirt"
#       config.vm.provider "virtualbox"
#     
#     
#       # Configure default box
#       config.vm.box = "chef/centos-7.1"
#       config.vm.provider "libvirt" do |v, override|
#         override.vm.box = "dliappis/centos7minlibvirt"
#         #override.vm.box = "GeeMedia/CentOS-7.1"
#       end
#     
#     #  config.vm.provider "libvirt" do |libvirt|
#     #    libvirt.connect_via_ssh = true
#     #    libvirt.host = arch["hyp_host"]
#     #    libvirt.username = arch["hyp_username"]
#     #    libvirt.id_ssh_key_file = arch["hyp_key"]
#     #  end
#     
#     
#     # VirtualBox settings:
#       #
#       # Vagrant.configure("2") do |config|
#       #   config.vm.network "private_network", ip: "192.168.50.4",
#       #     virtualbox__intnet: "mynetwork"
#       #   config.vm.network "private_network", ip: "192.168.50.4", nic_type: "virtio"
#       # end
#       #
#       # config.vm.provider "virtualbox" do |v|
#       # 	v.name = "my_vm"
#       # 	v.linked_clone = true if Vagrant::VERSION =~ /^1.8/
#       # 	v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
#       # 	v.customize ["modifyvm", :id, "--memory", "512"]
#       # 	v.customize ["modifyvm", :id, "--pagefusion", "on"]
#       # 	v.customize ["modifyvm", :id, "--acpi", "on"]
#       # 	v.customize ["modifyvm", :id, "--cpus", "2"]
#       # 	v.customize ["modifyvm", :id, "--cpuhotplug", "2"]
#       # 	v.customize ["modifyvm", :id, "--pae", "on"]
#       # 	v.customize ["modifyvm", :id, "--largepages", "on"]
#       # 	v.customize ["modifyvm", :id, "--guestmemoryballoon", "128"]
#       # 	v.customize ["modifyvm", :id, "--pae", "on"]
#       # end
#       #
#       #
#       # Docker:
#       # Vagrant.configure("2") do |config|
#       # 	# v2 configs...
#       #
#       #   config.vm.provider "docker" do |d|
#       #   	  # Required (at least one of them)
#       #       d.image = "foo/bar"
#       #       d.build_dir = "."
#       #
#       #       # optional
#       #       build_args = 
#       #       cmd = 
#       #       create_args= 
#       #       env = 
#       #       expose = 
#       #       link = 
#       #       force_host_vm = 
#       #       pull = false
#       #       remains_running = true
#       #       stop_timeout = 30
#       #       volumes = 
#       #
#       #       email =
#       #       username =
#       #       password = 
#       #       auth_server = 
#       #       dockerfile = 
#       #
#       #   end
#       # end
#       # config.vm.provider "docker" do |d|
#       #     d.image = "vm_name of the container"
#       #       end
#       #
#       # Libvirt:
#       # memory
#       # cpus
#       # nested
#       # volume_cache = unsafe
#       # keymap = 
#       # machine_virtual_size = 20G
#       # libvirt.random :model => 'random'
#       #
#       # net config:
#       # 	:libvirt__network_name:
#       # 	:libvirt__netmask
#       # 	:libvirt__host_ip
#       # 	:libvirt__forward_mode: nat
#       # 	:libvirt__forward_device
#       # 	:libvirt__guest_ipv6: no
#       # 	:model_type = virtio
#       #
#     
#     
#       # Turn off shared folders
#       config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", disabled: true
#     
#     
#       # Enable ansible provisionning
#       config.vm.provision "ansible" do |ansible|
#         ansible.sudo = true
#         ansible.extra_vars = { ansible_ssh_user: 'vagrant' }
#         #ansible.playbook = "../../deployment/rcp_deploy_arch.yml"
#         ansible.host_key_checking = false
#         ansible.playbook = arch["ansible_playbook"]
#         ansible.groups = arch["ansible_groups"]
#       end
#     
#       # Load each boxes
#       arch["boxes"].each do |name, opts|
#         config.vm.define name + arch["boxes_suffix"] do |config|
#     
#           # Define VM
#           config.vm.hostname = name + arch["boxes_suffix"]
#           config.vm.network :private_network, :ip => opts["ipaddr"], :libvirt__netmask => opts["ipnetmask"]
#     
#           # Define VM properties (LibVirt)
#           config.vm.provider "libvirt" do |l|
#             l.memory = opts["mem"]
#             l.cpus = opts["cpu"]
#           end
#     
#           # Define VM properties (VirtualBox)
#           config.vm.provider "virtualbox" do |v|
#             v.customize ["modifyvm", :id, "--memory", opts[:mem]]
#             v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
#           end
#     
#           # Change the default interface of the VM
#     #      config.vm.provision "shell", inline: "ip route delete default 2>&1 >/dev/null || true; ip route add default via " + opts["ipgateway"]
#     #      config.vm.provision "shell", inline: "sed -i 's/DEFROUTE=.*/DEFROUTE=no/' /etc/sysconfig/network-scripts/ifcfg-eth0"
#     #      config.vm.provision "shell", inline: "grep 'device=eth0' /etc/sysconfig/network-scripts/ifcfg-eth0 || echo device=eth0 >> /etc/sysconfig/network-scripts/ifcfg-eth0"
#     #      config.vm.provision "shell", inline: "grep 'DEFROUTE=yes' /etc/sysconfig/network-scripts/ifcfg-eth1 || echo DEFROUTE=yes >> /etc/sysconfig/network-scripts/ifcfg-eth1"
#     #      config.vm.provision "shell", inline: "yum remove -q -y NetworkManager || echo 'NetworkManager already removed'"
#     
#         end
#       end
#     end
#     

#
#