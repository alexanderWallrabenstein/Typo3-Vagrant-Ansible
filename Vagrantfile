# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
settings = YAML.load_file 'sync/config/config.yml'


# Vagrantfile API/syntax version.
VAGRANTFILE_API_VERSION = "2"

Vagrant.require_version ">= 1.9"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  
  # VM Configuration
  # 
  config.vm.box = "#{settings['vm']['box']}"
  config.vm.hostname = "#{settings['vm']['hostname']}"
  config.vm.post_up_message = "#{settings['vm']['post_up_message']}"
  config.vm.box_check_update = settings['vm']['post_up_message']
  config.vm.box_version = "#{settings['vm']['box_version']}"


  # Provider Configuration
  # 
  config.vm.provider "virtualbox" do |v|
    v.memory = settings['vm']['provider']['memory']
    v.cpus = settings['vm']['provider']['cpus']
    v.name = "#{settings['vm']['provider']['name']}"
    v.gui = settings['vm']['provider']['gui']

    settings['vm']['provider']['customize'].each do |cfg|
      v.customize ["#{cfg['command']}", :id, "#{cfg['option']}", "#{cfg['value']}"]
    end
  end


  # Network Configuration
  #   
  if settings['vm']['network']['private_network']['ip'] == "dhcp"
    config.vm.network "private_network", type: "dhcp"
  else
    config.vm.network :private_network, ip: "#{settings['vm']['network']['private_network']['ip']}"
    settings['vm']['network']['forwarded_port'].each do |cfg|
      config.vm.network "forwarded_port", guest: cfg['guest'], host: cfg['host'], auto_correct: cfg['auto_correct']
    end
  end


  # Shell Configuration
  # 
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'" # avoids 'stdin: is not a tty' error


  # Sync Folder Configuration
  #   
  settings['vm']['synced_folder'].each do |folder|
    
    # if not specified, set default value
    folder['mount_options'] = !folder['mount_options'].nil? ? folder['mount_options'] : ["dmode=777","fmode=777"]
    folder['disabled'] = !folder['disabled'].nil? ? folder['disabled'] : false

    config.vm.synced_folder folder['source'], folder['target'], id: folder['id'],
      mount_options: folder['mount_options'],
      disabled: folder['disabled'] 
  end
  

  # Configure Provisioning
  # 
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "#{settings['ansible']['playbook']}"
    ansible.groups = {
      "webserver" => ["default"],
      "dbserver" => ["default"],
    }
    ansible.tmp_path = "#{settings['ansible']['tmp_path']}"
    ansible.version = settings['ansible']['version']
    ansible.install = settings['ansible']['install']
    ansible.limit = "#{settings['ansible']['limit']}"
  end

end
