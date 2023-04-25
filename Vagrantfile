# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => { # задаем параметры и конфигурацию сетевых адаптеров для inetRouter
        :box_name => "centos/7",
        :vm_name => "inetRouter",
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.56.10', adapter: 8},
                ]
  },
  
  :centralRouter => { # задаем параметры и конфигурацию сетевых адаптеров для centralRouter
        :box_name => "centos/7",
        :vm_name => "centralRouter",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
                   {ip: '192.168.255.9', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                   {ip: '192.168.255.5', adapter: 7, netmask: "255.255.255.252", virtualbox__intnet: "office2-central"},
                   {ip: '192.168.56.11', adapter: 8},
                ]
  },
  
  :centralServer => { # задаем параметры и конфигурацию сетевых адаптеров для centralServer
        :box_name => "centos/7",
        :vm_name => "centralServer",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.56.12', adapter: 8},
                ]
  },

  :office1Router => { # задаем параметры и конфигурацию сетевых адаптеров для office1Router
        :box_name => "centos/7",
        :vm_name => "office1Router",
        :net => [
                   {ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                   {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "dev1-net"},
                   {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test1-net"},
                   {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"},
                   {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "office1-net"},
                   {ip: '192.168.56.20', adapter: 8},
                ]
  },

  :office1Server => { # задаем параметры и конфигурацию сетевых адаптеров для office1Server
        :box_name => "centos/7",
        :vm_name => "office1Server",
        :net => [
                   {ip: '192.168.2.130', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"},
                   {ip: '192.168.56.21', adapter: 8},
                ]
  },

  :office2Router => { # задаем параметры и конфигурацию сетевых адаптеров для office2Router
    :box_name => "centos/7",
    :vm_name => "office2Router",
    :net => [
               {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2-central"},
               {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},
               {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test2-net"},
               {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office2-net"},
               {ip: '192.168.56.30', adapter: 8},
            ]
  },

  :office2Server => { # задаем параметры и конфигурацию сетевых адаптеров для office2Server
        :box_name => "centos/7",
        :vm_name => "office2Server",
        :net => [
                   {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},
                   {ip: '192.168.56.31', adapter: 8},
                ]
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig| 

    config.vm.define boxname do |box| 

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "256"] 
        end

        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.ip_forward=1" >>/etc/sysctl.conf # Настройка маршрутизации транзитных пакетов (не изменяется после перезапуска) 
            touch /etc/sysconfig/network-scripts/route-eth1 # создаем файл (интерфейса eth1 - вся сеть) для хранения обраных маршрутов (не изменяется после перезапуска)
            SHELL

        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.ip_forward=1" >>/etc/sysctl.conf # Настройка маршрутизации транзитных пакетов (не изменяется после перезапуска) 
            touch /etc/sysconfig/network-scripts/route-eth5 # создаем файл (интерфейса eth5 - office1) для хранения обраных маршрутов (не изменяется после перезапуска)
            touch /etc/sysconfig/network-scripts/route-eth6 # создаем файл (интерфейса eth6 - office2) для хранения обраных маршрутов (не изменяется после перезапуска)
            SHELL

        when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.ip_forward=1" >>/etc/sysctl.conf # Настройка маршрутизации транзитных пакетов (не изменяется после перезапуска) 
            SHELL

        when "office1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "Privet" # контрольная метка :)
            SHELL

        when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.ip_forward=1" >>/etc/sysctl.conf # Настройка маршрутизации транзитных пакетов (не изменяется после перезапуска) 
            SHELL
  
        when "office2Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "Privet" # контрольная метка :)
            SHELL

        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
          touch /etc/sysconfig/network-scripts/route-eth1 # Настройка маршрутизации транзитных пакетов (не изменяется после перезапуска) 
            SHELL
        end

        box.vm.provision "ansible" do |ansible|  # вызываем playbook для развертывания нужной инфраструктуры на машинах
          ansible.verbose = "v"
          ansible.playbook = "lan.yml" # файл playbook
        end
      end

  end
  
  
end