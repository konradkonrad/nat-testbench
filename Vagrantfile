# -*- mode: ruby -*-
# vi: set ft=ruby :

# VAGRANTFILE for creating a NAT testing setup.
# - all clients share devices on the "public internet" in the `192.168.42.*` network.
# - there is a STUN server on `192.168.42.10` and `192.168.42.20`, all VMs have host entries
#   `stun.server.local` and `stun2.server.local` that point to above addresses.
# - NAT clients have a "LAN" IP in the `10.10.4x.2` range, that is NAT-ted to their "internet" IP
#   by various methods (fullcone, restrictedcone, portrestrictedcone, symmetric).
# - NAT clients have /etc/hosts entries `my-lan-ip` and `my-wan-ip` that point to their "LAN" and
#   "internet" IPs.
#
#   Usage:
#     vagrant up stun  # for starting the STUN server
#     vagrant up <method>  # for starting a specific NAT method, e.g.
#     vagrant up fullcone  # for starting the 'fullcone' NAT client


## GENERAL PROVISIONING SECTION
# adjust this, if you want to install packages on all VMs
Vagrant.configure("2") do |config|
  # root
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y python-virtualenv python-dev
    echo "192.168.42.10 stun.server.local" >> /etc/hosts
    echo "192.168.42.20 stun2.server.local" >> /etc/hosts
    echo "192.168.42.100 fullcone" >> /etc/hosts
    echo "192.168.42.110 restrictedcone" >> /etc/hosts
    echo "192.168.42.120 portrestrictedcone" >> /etc/hosts
    echo "192.168.42.130 symmetric" >> /etc/hosts
  SHELL
  # user
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    virtualenv ~/.venv
    source ~/.venv/bin/activate
    pip install -U pip
    pip install -U setuptools
    pip install ipython
    pip install pystun
    echo "source ~/.venv/bin/activate" >> ~/.bashrc
  SHELL
## /GENERAL PROVISIONING SECTION

  config.vm.define "stun" do |stun|
      stun.vm.box = "ubuntu/trusty64"
      stun.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get -y install stun
        stund -v -h 192.168.42.10 -a 192.168.42.20 -b &> stun.log
      SHELL
      stun.vm.network "private_network", ip: "192.168.42.10"
      stun.vm.network "private_network", ip: "192.168.42.20"
  end
  # these follow https://wiki.asterisk.org/wiki/display/TOP/NAT+Traversal+Testing with some
  # fixes/adjustments
  # see also: http://lists.netfilter.org/pipermail/netfilter/2007-April/068463.html
  # in our case private is 'eth2' and public is 'eth1'
  config.vm.define "fullcone" do |fullcone|
      fullcone.vm.box = "ubuntu/trusty64"
      fullcone.vm.network "private_network", ip: "192.168.42.100"  # "internet"
      fullcone.vm.network "private_network", ip: "10.10.44.2"  # "LAN"
      fullcone.vm.provision "shell", inline: <<-SHELL
        iptables -t nat -A POSTROUTING -o eth1 -j SNAT --to-source 192.168.42.100
        iptables -t nat -A PREROUTING -i eth1 -j DNAT --to-destination 10.10.44.2
        echo "10.10.44.2 my-lan-ip" >> /etc/hosts
        echo "192.168.42.100 my-wan-ip" >> /etc/hosts
      SHELL
  end
  config.vm.define "restrictedcone" do |restrictedcone|
      restrictedcone.vm.box = "ubuntu/trusty64"
      restrictedcone.vm.network "private_network", ip: "192.168.42.110"  # "internet"
      restrictedcone.vm.network "private_network", ip: "10.10.45.2"  # "LAN"
      restrictedcone.vm.provision "shell", inline: <<-SHELL
        iptables -t nat -A POSTROUTING -o eth1 -p tcp -j SNAT --to-source 192.168.42.110 
        iptables -t nat -A POSTROUTING -o eth1  -p udp -j SNAT --to-source 192.168.42.110 
        iptables -t nat -A PREROUTING -i eth1 -p tcp -j DNAT --to-destination 10.10.45.2 
        iptables -t nat -A PREROUTING -i eth1 -p udp -j DNAT --to-destination 10.10.45.2 
        iptables -A INPUT -i eth1 -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT 
        iptables -A INPUT -i eth1 -p udp -m state --state ESTABLISHED,RELATED -j ACCEPT 
        iptables -A INPUT -i eth1 -p tcp -m state --state NEW -j DROP 
        iptables -A INPUT -i eth1 -p udp -m state --state NEW -j DROP 
        echo "10.10.45.2 my-lan-ip" >> /etc/hosts
        echo "192.168.42.110 my-wan-ip" >> /etc/hosts
      SHELL
  end
  config.vm.define "portrestrictedcone" do |portrestrictedcone|
      portrestrictedcone.vm.box = "ubuntu/trusty64"
      portrestrictedcone.vm.network "private_network", ip: "192.168.42.120"  # "internet"
      portrestrictedcone.vm.network "private_network", ip: "10.10.46.2"  # "LAN"
      portrestrictedcone.vm.provision "shell", inline: <<-SHELL
        iptables -t nat -A POSTROUTING -o eth1 -j SNAT --to-source 192.168.42.120
        echo "10.10.46.2 my-lan-ip" >> /etc/hosts
        echo "192.168.42.120 my-wan-ip" >> /etc/hosts
      SHELL
  end
  config.vm.define "symmetric" do |symmetric|
      symmetric.vm.box = "ubuntu/trusty64"
      symmetric.vm.network "private_network", ip: "192.168.42.130"  # "internet"
      symmetric.vm.network "private_network", ip: "10.10.47.2"  # "LAN"
      symmetric.vm.provision "shell", inline: <<-SHELL
        echo "1" > /proc/sys/net/ipv4/ip_forward
        iptables --flush
        iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE --random
        iptables -A FORWARD -i eth1 -o eth2 -m state --state RELATED,ESTABLISHED -j ACCEPT
        iptables -A FORWARD -i eth2 -o eth1 -j ACCEPT
        echo "10.10.47.2 my-lan-ip" >> /etc/hosts
        echo "192.168.42.130 my-wan-ip" >> /etc/hosts
      SHELL
  end
end
