# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Detect host OS for different folder share configuration
module OS
    def OS.windows?
        (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
    end

    def OS.mac?
        (/darwin/ =~ RUBY_PLATFORM) != nil
    end

    def OS.unix?
        !OS.windows?
    end

    def OS.linux?
        OS.unix? and not OS.mac?
    end
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "bento/centos-7.3"

  config.vm.provider "parallels"
  config.vm.provider "virtualbox"

  # Check available Plugins
  if OS.windows?
      if !Vagrant.has_plugin?('vagrant-winnfsd')
          puts "The vagrant-winnfsd plugin is required. Please install it with \"vagrant plugin install vagrant-winnfsd\""
          exit
      end
  end

  if Vagrant.has_plugin?('vagrant-vbguest')
      config.vbguest.auto_update = true
  end

  config.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.customize ['modifyvm', :id, '--memory', 2048]
      vb.customize ["modifyvm", :id, "--cpus", 2]
      vb.customize ["modifyvm", :id, "--name", "vagrant-centos7-fog"]
  end

  config.vm.provider "parallels" do |v|
      v.memory = 2048
      v.cpus = 2
  end

  # Configure the VM
  config.vm.hostname = 'fog-test'
  config.vm.network :forwarded_port, host: 80, guest: 80      # website
  config.vm.network :forwarded_port, guest: 3306, host: 3306  # mariadb
  config.vm.network :forwarded_port, guest: 20, host: 20  # ftp
  config.vm.network :forwarded_port, guest: 21, host: 21  # ftp
  config.vm.network :forwarded_port, guest: 22, host: 22  # ssh
  config.vm.network :forwarded_port, guest: 69, host: 69  # tftp
  config.vm.network :forwarded_port, guest: 443, host: 443  # https
  config.vm.network :forwarded_port, guest: 111, host: 111  # portmap
  config.vm.network :forwarded_port, guest: 2049, host: 2049  # NFS
  config.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)", ip: "192.168.1.110"

  # Configre shared folder
  if OS.windows?
    config.vm.synced_folder ".", "/vagrant", type: "nfs"
  else
    config.vm.synced_folder ".", "/vagrant", :owner => "vagrant", :group => "vagrant"
  end

  # Add the ssh keys before provisioning
  config.vm.provision "shell" do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      mkdir /root/.ssh
      echo "#{ssh_pub_key}" >> /home/vagrant/.ssh/authorized_keys
      echo "#{ssh_pub_key}" >> /root/.ssh/authorized_keys
    SHELL
  end

  # Run the provisioning - I'll add this back in later
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "ansible/playbook.yml"
    ansible.become = true
  end


end
