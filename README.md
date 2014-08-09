Vagrant base Box for CentOS 6.5
=========
I created a base Box for Vagrant based on CentOS 6.5 Minimal Install.

### Features

  - root password is "vagrant"
  - Default vagrant "insecure" key
  - Added EPEL repository
  - Added PUPPET repository
  - Installed "Development Tools"
  - Installed wget, man, bind-utils
  - Installed puppet, facter
  - Installed Virtualbox Guest Additions

### PreConditions (Windows)

  - Vagrant (1.6.3)
  - [Vagrant SSH-"insecure"-Key](https://github.com/mitchellh/vagrant/tree/master/keys)
  - VirtualBox (4.3.12)
  - Putty or Git-Bash

Create VM in VirtualBox without GUI
--------------
To begin, we have to configure a new VM in VirtualBox.

    Name: vagrant-{distro}-{version}
    Type: Linux
    Version: {distro} (64bit)
    Memory: 1024MB 
    CPU: 1
    Virtual Disk
        VMDK (Dynamic)
        Size: ~50GB
    Disable Audio
    Disable USB
    Set Network Adapter 1 to NAT with Port Forwarding
        Name: SSH
        Protocol: TCP
        Host Port: 2222
        Guest Port: 22

Create VM in VirtualBox with GUI
--------------
To begin, we have to configure a new VM in VirtualBox.

    Name: vagrant-{distro}-{version}
    Type: Linux
    Version: {distro} (64bit)
    Memory: 2048MB++ 
    CPU: 1 or 2
    32MB Video RAM with 3D Acceleration to support GUI Desktop
    Virtual Disk
        VMDK (Dynamic)
        Size: ~50GB
    Disable Audio
    Disable USB
    Set Network Adapter 1 to NAT with Port Forwarding
        Name: SSH
        Protocol: TCP
        Host Port: 2222
        Guest Port: 22

Installation
--------------

Download [CentOS 6.5](http://mirror.centos.org/centos-6/6.5/isos/)
	
    Host Name: vagrant-{distro}-{version-nr}
    Root Password: vagrant

VM Config
--------------
#### Network interface
    vi /etc/sysconfig/network-scripts/ifcfg-eth0
    ONBOOT= yes
    [ESC] :wq
    
    $ service network restart
    
##### (optional) Connect via Putty or Git-Bash 
    # Putty
    IP 127.0.0.1
    Port 2222 
	
	# Git-Bash
	ssh -p 2222 root@localhost
	or
	ssh -p 2222 root@127.0.0.1
    
#### Create User
    $ useradd -m -c "vagrant" vagrant
    $ passwd vagrant

#### Add vagrant User to Sudoers
    $ visudo
    # Add the following line to the end of the file.
    vagrant 	ALL=(ALL) 	NOPASSWD:ALL
    [ESC] :wq

#### Add TTY and SSH-Auth (see Error solving at the end)
	$ visudo
	
	# Search "Default requiretty" and ignore it with #
	# Default requiretty

	# Search "Defaults env_keep" and add this new line at the end
	Defaults env_keep+="SSH_AUTH_SOCK"
	[ESC] :wq

#### (optional) Disable Firewall (iptables) and IPv6
	$ service iptables stop
	$ chkconfig iptables off

	$ vi /etc/sysctl.conf
 	# Add the following line to the end of the file.
	# disable ipv6
	net.ipv6.conf.all.disable_ipv6 = 1
	net.ipv6.conf.default.disable_ipv6 = 1
	[ESC] :wq
 
	$ echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
	$ echo 1 > /proc/sys/net/ipv6/conf/default/disable_ipv6
    
#### Repos hinzufügen
    # EPEL
    $ rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

    # PUPPET
    $ rpm -ivh http://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-7.noarch.rpm

#### Update Packages
    $ yum update -y
    $ shutdown -r now
    
#### Packages installieren
    $ yum groupinstall "Development Tools"
    $ yum install kernel-devel kernel-headers wget man bind-utils.x86_64
    $ yum install dkms
    $ yum install puppet facter

#### OpenSSH configuration
    $ vi /etc/ssh/sshd_config
    Port 22
    PubKeyAuthentication yes
    AuthorizedKeysFile ~/.ssh/authorized_keys
    PermitEmptyPasswords no
    UseDNS no
	
    $ service sshd restart

#### Mount VirtualBox Guest Additions
    $ mkdir /media/VirtualBoxGuestAdditions
    $ mount -r /dev/cdrom /media/VirtualBoxGuestAdditions

#### Install VirtualBox Guest Additions
    $ cd /media/VirtualBoxGuestAdditions
    $ ./VBoxLinuxAdditions.run

    $ shutdown -r now
    
#### Vagrant OpenSSH-Keys
    $ su vagrant
    $ cd
    $ mkdir .ssh
    $ cd .ssh/
    $ wget https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O authorized_keys
    $ chmod 0700 .ssh/
    $ chmod 0600 .ssh/authorized_keys
    $ chown -R vagrant .ssh/

#### Cleanup
	$ rm -rf /tmp/*
	$ rm -f /var/log/wtmp
	$ rm -f /var/log/btmp
	$ yum clean all
	$ history -c

	$ shutdown -h now

#### Create the Box
    # cd to Dir to Export Base-Box
    $ vagrant package --base vagrant-{distro}-{version}
    
    ### SAMPLE
    # cd to Dir to Export Base-Box
    $ vagrant package --base vagrant-centos-6.5
    $ mv package.box centos65-x86_64-20140707.box

#### Testing & Working with the Box
    $ vagrant box add {boxname} package.box
    $ vagrant init {boxname}
    $ vagrant up

    ### SAMPLE
    $ vagrant box add centos65 centos65-x86_64-20140707.box
    $ mkdir test_env
    $ cd test_env/
    $ vagrant init centos65
    $ vagrant up
    $ vagrant ssh

### <span style="color:red">Error solving (sudo: sorry, you must have a tty to run sudo)</span>
    # Edit vagrantfile from actVM
    add Line 
	    => config.ssh.pty = true 

### Vagrantfile (with GUI and more memory)
	config.vm.provider "virtualbox" do |vb|
		vb.gui = true
		vb.customize ["modifyvm", :id, "--memory", "4096"]
  	end

License
--------------

MIT
