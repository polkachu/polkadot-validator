# -*- mode: ruby -*-
# vi: set ft=ruby :
BOX_IMAGE = "ubuntu/hirsute64"

Vagrant.configure("2") do |config|
  instances = [
    {"name" => "kusama1", "ip" => "192.168.1.10"},
    {"name" => "polkadot1", "ip" => "192.168.1.20"},
    {"name" => "monitor", "ip" => "192.168.1.30"},
  ]

  instances.each do |box|
    config.vm.define box['name'] do |cfg|
      cfg.vm.box = BOX_IMAGE
      cfg.vm.hostname = box['name']
      cfg.vm.network "private_network", ip: box['ip']
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
      cfg.vm.provision 'shell', inline: "echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys", privileged: false
    end
  end
end
