# -*- mode: ruby -*-
# vi: set ft=ruby :

# General project settings
#################################

  # IP Address for the host only network, change it to anything you like
  # but please keep it within the IPv4 private network range
  ip_address = "10.1.4.120"

  # The project name is base for directories, hostname and alike
  project_name = "ic5"

  #Show VirtualBox GUI for the VM
  show_vbox_gui = true

# Shell commands to be executed by shell provisioner below
# Install TDI and TDS silently
##########################################################

$script = <<SCRIPT
echo "*shell-prov: Running 'sudo yum update' and installing pre-req packages not present in Vagrant Box"

sudo echo "domain sandbox" > resolv.conf
sudo echo "search sandbox" >> resolv.conf
sudo echo "nameserver 10.1.2.1" >> resolv.conf
sudo cp resolv.conf /etc/resolv.conf

sudo yum update
sudo cp /vagrant/dvd.repo /etc/yum.repos.d
sudo yum install libgcc_s.so.1 gtk2-engines.i686 libXtst.i686 -y

echo "*shell-prov: Installing IIM V1.7.2"
sudo /ic5/ic5-nfa/IIM_V1.7.2/installc -acceptLicense

echo "*shell-prov: Installing WebSphere Application Server (Deployment Manager) V8.5.5"
sudo /opt/IBM/InstallationManager/eclipse/launcher -acceptLicense -input /ic5/ic5-nfa/WAS855.response -silent

echo "*shell-prov: Creating Deployment Manager Profile"
cd /opt/IBM/WebSphere/DeploymentManager
sudo ./bin/manageprofiles.sh -create -templatePath profileTemplates/management -serverType DEPLOYMENT_MANAGER -isDefault -enableAdminSecurity true -adminPassword wasadmin -adminUserName wasadmin

echo "*shell-prov: Installing WebSphere Application Server V8.5.5"
sudo /opt/IBM/InstallationManager/eclipse/launcher -acceptLicense -input /ic5/ic5-nfa/WAS855App.response -silent

echo "*shell-prov: Creating Application Server Profile"
sudo /opt/IBM/WebSphere/AppServer/bin/manageprofiles.sh -create -templatePath /opt/IBM/WebSphere/AppServer/profileTemplates/default

echo "*shell-prov: Federating Application Server profile into Deployment Manager"
sudo /opt/IBM/WebSphere/DeploymentManager/profiles/Dmgr01/bin/startManager.sh
sudo /opt/IBM/WebSphere/AppServer/profiles/AppSrv01/bin/addNode.sh ic5 -username wasadmin -password wasadmin

echo "*shell-prov: Installing IBM HTTP Server V8.5.5 and WebSphere Application Server Plugin"
sudo /opt/IBM/InstallationManager/eclipse/launcher -acceptLicense -input /ic5/ic5-nfa/HTTP.response -silent

echo "*shell-prov: Configuring WebSphere Application Server Plugin"
sudo /opt/IBM/WebSphere/Toolbox/WCT/wctcmd.sh -tool pct -createDefinition -defLocName ic5Plugin -defLocPathname /opt/IBM/WebSphere/Plugins -response /ic5/ic5-nfa/pct_responsefile.txt
SCRIPT

# Mount RHEL ISO file under Mac OSX host /Volumes
#################################
system 'hdiutil mount ~/Downloads/RHEL6.4-20130130.0-Server-x86_64-DVD1.iso'


# Vagrant configuration
#################################

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.dnsmasq.domain = project_name + '.sandbox'
  # overwrite default location for /etc/dnsmasq.conf
  # brew_prefix = `brew --prefix`.strip
  config.dnsmasq.dnsmasqconf = '/usr/local/etc/dnsmasq.conf'

  # command for reloading dnsmasq after config changes
  config.dnsmasq.reload_command = 'sudo launchctl unload /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist; sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist'
  config.dnsmasq.keep_resolver_on_destroy = true
  config.dnsmasq.ip = ip_address
  

  config.vm.box = "parallels/rhel-6.4-x86_64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  dir1 = "/" + project_name + "/ic5-nfa"
  
  config.vm.synced_folder "../nfa.ic5", dir1
  config.vm.synced_folder "/Volumes/RHEL_6.4 x86_64", "/" + project_name + "/rhel"

  # Define a box wiht a hostname 
  # the path on the host to the actual folder. The second argument is
  config.vm.define project_name do |node|
    
    # Configure box hostname and private network for host<>box communication
    node.vm.hostname = project_name
    node.vm.network :private_network, ip: ip_address

    config.vm.provider "parallels" do |v|
      v.memory = 1024
      v.cpus = 2
    end
    
    # Enable VirtualBox GUI for the VM
    config.vm.provider "virtualbox" do |vb|
      
      # Don't boot with headless mode
      vb.gui = show_vbox_gui
  
      # Use VBoxManage to customize the VM. For example to change memory:
      vb.customize ["modifyvm", :id, "--memory", "2048"]
      vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end
  
  config.vm.provision :shell, :inline => $script


  # config.vm.provision "shell" do |sh|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { mysql_password: "foo" }
  # end


  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { mysql_password: "foo" }
  # end

end
