# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  CONTROLLER_CLUSTER_IP="10.3.0.1"
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL

  def etcdIP(num)
    return "172.17.4.#{num+50}"
  end

  def controllerIP(num)
    return "172.17.4.#{num+100}"
  end

  def workerIP(num)
    return "172.17.4.#{num+200}"
  end

  # Generate root CA
  system("mkdir -p ssl && ./lib/init-ssl-ca ssl") or abort ("failed generating SSL artifacts")

  # Generate admin key/cert
  system("./lib/init-ssl ssl admin kube-admin") or abort("failed generating admin SSL artifacts")


  (1..1).each do |i|
    config.vm.define "etcd-#{i}" do |etcd|
      etcd.vm.hostname = "etcd-#{i}"
      etcd.vm.network :private_network, ip: etcdIP(i)

      # etcd.vm.provision "shell", inline: "curl -L https://github.com/coreos/etcd/releases/download/v2.3.8/etcd-v2.3.8-linux-amd64.tar.gz -o /tmp/etcd-v2.3.8-linux-amd64.tar.gz"
      # etcd.vm.provision "shell", inline: "mkdir -p /tmp/test-etcd && tar xzvf /tmp/etcd-v2.3.8-linux-amd64.tar.gz -C /tmp/test-etcd --strip-components=1"
      # etcd.vm.provision "shell", inline: "sudo mv /tmp/test-etcd/etcd* /usr/bin/"
      # etcd.vm.provision "shell", inline: "sudo mkdir -p /var/lib/etcd"
      # etcd.vm.provision "shell", inline: "etcd --version"
    end
  end

  def provisionMachineSSL(machine,certBaseName,cn,ipAddrs)
    tarFile = "ssl/#{cn}.tar"
    ipString = ipAddrs.map.with_index { |ip, i| "IP.#{i+1}=#{ip}"}.join(",")
    system("./lib/init-ssl ssl #{certBaseName} #{cn} #{ipString}") or abort("failed generating #{cn} SSL artifacts")
    puts 'hey'
    machine.vm.provision :file, :source => tarFile, :destination => "/tmp/ssl.tar"
    puts 'first done'
    machine.vm.provision :shell, :inline => "mkdir -p /etc/kubernetes/ssl && tar -C /etc/kubernetes/ssl -xf /tmp/ssl.tar", :privileged => true
  end

  controllerIPs = [*1..2].map{ |i| controllerIP(i) } <<  CONTROLLER_CLUSTER_IP
# etcdIPs = [*1..3].map{ |i| etcdIP(i) }
# initial_etcd_cluster = etcdIPs.map.with_index{ |ip, i| "e#{i+1}=http://#{ip}:2380" }.join(",")
# etcd_endpoints = etcdIPs.map.with_index{ |ip, i| "http://#{ip}:2379" }.join(",")

  (1..1).each do |i|
    config.vm.define vm_name = "c%d" % i do |controller|

      # env_file = Tempfile.new('env_file', :binmode => true)
      # env_file.write("ETCD_ENDPOINTS=#{etcd_endpoints}\n")
      # env_file.close

      controller.vm.hostname = vm_name

      controllerIP = controllerIP(i)
      controller.vm.network :private_network, ip: controllerIP

      # Each controller gets the same cert
      provisionMachineSSL(controller,"apiserver","kube-apiserver-#{controllerIP}",controllerIPs)

      # controller.vm.provision :file, :source => env_file, :destination => "/tmp/coreos-kube-options.env"
      # controller.vm.provision :shell, :inline => "mkdir -p /run/coreos-kubernetes && mv /tmp/coreos-kube-options.env /run/coreos-kubernetes/options.env", :privileged => true

      # controller.vm.provision :file, :source => CONTROLLER_CLOUD_CONFIG_PATH, :destination => "/tmp/vagrantfile-user-data"
      # controller.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
    end
  end


end
