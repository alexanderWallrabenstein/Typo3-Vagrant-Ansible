# Typo3-Vagrant-Ansible

This repository aims to create a reproducible Typo3 development environment using [Vagrant](http://www.vagrantup.com/) and [Ansible](https://www.ansible.com/). 

You can either create a new, clean Typo3 installation or import an existing setup. This allows you to save time and trouble creating a working development environment. Since the setup (except data) is under version control, you can rebuild the entire environment reliably and without manual intervention!

## Installation
_By default, a new and clean installation of Typo3 will be created. Here's how you can [import an existing Typo3 setup](#2-create-a-typo3-installation-but-import-data-from-an-existing-setup)_

### Requirements
* [VirtualBox](https://www.virtualbox.org/) - Version 5.* (free)
* [Vagrant](http://www.vagrantup.com/) - Version 1.9.* (free)

### Using git
* open a terminal
* clone this repository: `git clone https://github.com/alexanderWallrabenstein/Typo3-Vagrant-Ansible.git`
* change into the new folder: `cd Typo3-Vagrant-Ansible` 
* Run `vagrant up`

### Without git
* Download the latest version [Typo3-Vagrant-Ansible](https://github.com/alexanderWallrabenstein/Typo3-Vagrant-Ansible/archive/master.zip)
* Extract the zip file
* open a terminal
* change into the new folder: `cd Typo3-Vagrant-Ansible`
* Run `vagrant up` 

_The very first run of `vagrant up` may take a few minutes since the basebox, Typo3 source code and all required packages need to be downloaded and installed._

**After** `vagrant up`:

Your virtual machine is now ready to use! It's fully booted and provisioned.

You can:
* log into your VM: `vagrant ssh`
* view it in your browser [http://localhost:8080](http://localhost:8080)

## High-level structure of important files and directories
```
Typo3-Vagrant-Ansible
├───Vagrantfile
└───sync
    ├───ansible
    │   ├───site.yml
    │   └───roles
    │       ├───common
    │       │   ├───handlers
    │       │   └───tasks
    │       ├───db
    │       │   ├───handlers
    │       │   └───tasks
    │       └───web
    │           ├───handlers
    │           └───tasks
    ├───config
    │   ├───.htcaccess
    │   ├───ansible.cfg
    │   ├───config.yml
    │   └───vhost80.conf
    └───typo3
        ├───fileadmin (UC2)
        ├───typo3conf (UC2)
        ├───uploads (UC2)
        ├───typo3_db.sql (UC2)
        └───typo3_src (A)

(UC2) indicates that this file or directory is only relevant in Use Case 2 (see below)
(A) indicates that this file or directory is created or downloaded automatically
```

<dl>
<dt>Vagrantfile</dt>
    <dd>'Template' for the configuration of your VM</dd>
<dt>sync/</dt>
    <dd>Subdirectories are mounted into the VM, see config.yml</dd>
<dt>sync/ansible/</dt>
    <dd>Contains all files and directories used by Ansible</dd>
<dt>sync/config</dt>
    <dd>Contains config files used during setup and provisioning</dd>
<dt>sync/config/config.yml</dt>
    <dd><em>Main configuration file</em>: this file contains all configuration parameters used by Vagrant and Ansible</dd>
<dt>typo3/</dt>
    <dd>Contains main Typo3 directories if you want to 'import' an existing Typo3 installation. Store your sql dump here. Ansible will store the downloaded Typo3 source archive file here for reuse after the initial provisioning. More information in 'The sync/typo3 directory'</dd>
</dl>

## Vagrant
[Vagrant](https://www.vagrantup.com/) is a tool to configure reproducible environments. It manages and defines the basic virtual machine(s). The 'Vagrantfile' stores the setup and configuration of the VM itself and the provider, in this case VirtualBox. Furthermore, it is used to specify a provisioner, which takes over after Vagrant has created the basic VM. This setup uses Ansible as provisioner of choice. See further [Documentation](https://www.vagrantup.com/docs/).

Useful commands:
* start the machine `vagrant up` (create and provision during initial run)
* destroy the machine `vagrant destroy`
* log into the machine via ssh `vagrant ssh`
* print ssh access information `vagrant ssh-config`
* reload and provision the machine `vagrant reload --provisioning`
* provision the machine `vagrant provision`
* shut down the machine `vagrant halt`

_Execute all commands at the root of this repository where the [Vagrantfile](Vagrantfile) resides._

## Ansible
[Ansible](https://www.ansible.com/) is a configuration management and orchestration tool. It automates the setup and deployment of a system according to configuration files. The syntax is simple, declarative [YAML](http://yaml.org/). 
The main [playbook](https://docs.ansible.com/ansible/playbooks.html) is [site.yml](sync/ansible/site.yml). It controls the overall setup by applying the three different roles (common, db, web) to the host managed by Vagrant. Each role consists of tasks and handlers. The _main.yml_ file in each directory is included automatically by Ansible. See further [Documentation](https://docs.ansible.com/).

## Interplay of Vagrant and Ansible
Vagrant uses the [ansible_local](https://www.vagrantup.com/docs/provisioning/ansible_local.html) provisioner to configure and deploy the machine after boot. Per configuration in [config.yml](sync/config/config.yml), Vagrant installs Ansible on the VM, generates the [inventory file](https://docs.ansible.com/ansible/intro_inventory.html) based on information of the Vagrantfile and executes the main playbook automatically.

## Use cases
### 1) Create a clean Typo3 installation
* set in [config.yml](sync/config/config.yml) `general['clean_typo3_install']: true`
* open [http://localhost:8080](http://localhost:8080) in a browser 
* walk through the standard Typo3 installation process:
    - 1) System environment check. Click 'Continue'.
    - 2) Database connection
        + select: Type 'TCP/IP based connection'
        + enter username and password provided in [config.yml](sync/config/config.yml) `mysql['users']['typo3']`
    - 3) Select database  
        + choose: 'Use an existing empty database'
        + select the database name provided in [config.yml](sync/config/config.yml) `mysql['databases']['typo3_local']`
    - 4) Create user and import base data
        + set _username_ to _admin_ and enter a secure password
    - 5) Done! You can now log in to your Typo3 backend
    
### 2) Create a Typo3 installation but import data from an existing setup
* set in [config.yml](sync/config/config.yml) `general['clean_typo3_install']: false`
* create a sql dump of your existing Typo3 installation
    - on your existing system: `mysqldump -h {host} -u {user} -p{password} {databasename} > typo3_db.sql`
    - copy it in _sync/typo3/_ via _SFTP client_ or _scp_
    - check path in [config.yml](sync/config/config.yml) `mysql['sql_dump_path']`
* copy main Typo3 directories
    - copy _fileadmin_, _typo3conf_ and _uploads_ directory from your existing Typo3 installation to _sync/typo3/_ using a _SFTP client_ or _scp_ on UNIX systems `scp -r user@your.sever.com:/path/to/typo3/<folder> sync/typo3/<folder>`

_might be necessary after the provisioning_

* edit database parameters in _LocalConfiguration.php_
    - open _typo3conf/LocalConfiguration.php_ with a texteditor
    - set _database_, _host_, _password_, _username_ in 'DB' according to `mysql['users']['typo3']` and `mysql['databases']['typo3_local']` in [config.yml](sync/config/config.yml)
* adapt settings in Typoscript
    - login to your Typo3 backend, open the Typoscript template for your root page and set
        ```
        config.baseURL = localhost
        config.absRefPrefix = /
        ```


## The sync/typo3 directory

### Why this directory?
This directory is configured as a 'synced_folder' between your host and guest system in [config.yml](sync/config/config.yml). Hence, it's content will not be erased when the guest system will be destroyed or provisioned.

### Usage
#### Ansible caches Typo3 source code
Ansible will downloaded the Typo3 source code archive during the provisioning. To save time and bandwidth during any further run of the provisioning, it will store the downloaded Typo3 source code in this directory for reuse. It's kind of a 'cache'.

#### Store your Typo3 main directories here
As described in [Create a Typo3 installation but import data from an existing setup](#2-create-a-typo3-installation-but-import-data-from-an-existing-setup) you should store your _fileadmin_,_typo3conf_ and _uploads_ folder here, so that it can be either symlinked or copied to the 'DocumentRoot' `/var/www/site/htdocs`. You can set this option in the _config.yml_ file. In addition, place your sql dump in this directory, too.
In this use case, the _sync/typo3_ directory will again be used as a 'cache' for your data, which must 'survive' multiple runs of the Ansible provisioning.

## Troubleshooting
Both Vagrant and Ansible print their error output directly to your console. Additionally, you can inspect the log files within your VM in `/var/log`.

## Contribution
If you have any questions, ideas for improvement or you encountered a problem, please feel free to open an issue or contact me.

## License
The MIT License (MIT)

Copyright (c) 2017 Alexander Wallrabenstein <alexander.wallrabenstein@gmail.com
