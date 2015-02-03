# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  vm_ip = "192.168.59.103"
  vm_network = vm_ip.split('.')[0..2].join('.')
  config.vm.network "private_network", ip: vm_ip

  host_ip = `VBoxManage list hostonlyifs | grep IPAddress| grep #{vm_network}`.split()[1]
  if ! host_ip
    # Setup hostonly network
    vboxnet = /'.*'/.match(`VBoxManage hostonlyif create 2>/dev/null`).to_s
    puts vboxnet
    `VBoxManage hostonlyif ipconfig #{vboxnet} --ip 192.168.59.1 2>/dev/null`
    host_ip = "192.168.59.1"
  end



  config.vm.provider "virtualbox" do |v|
    # Pass the vboxnet ip as a Vbox guest prop so we can read it from the vm. 
    v.customize ["guestproperty", "set", :id, "/VirtualBox/HostInfo/HostOnlyIP", host_ip]

    host = RbConfig::CONFIG['host_os']

    # Give VM 1/2 system memory & access to all cpu cores on the host
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
      # sysctl returns Bytes and we need to convert to MB
      mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 2
    elsif host =~ /linux/
      cpus = `nproc`.to_i
      # meminfo shows KB and we need to convert to MB
      mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 2
    else # sorry Windows folks, I can't help you
      cpus = 2
      mem = 1024
    end
    
    v.customize ["modifyvm", :id, "--memory", mem]
    v.customize ["modifyvm", :id, "--cpus", cpus]
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "vagrant.yml"
  end
end
