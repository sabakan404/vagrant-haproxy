# -*- mode: ruby -*-
# vi: set ft=ruby :

$box_name = "CentOS-7.1.1503-x86_64"
$box_url = "https://github.com/holms/vagrant-centos7-box/releases/download/7.1.1503.001/CentOS-7.1.1503-x86_64-netboot.box"
$num_instances = 2

Vagrant.configure(2) do |config|

  (1..$num_instances).each do |i|
    config.vm.define vm_name = "%s-%02d" % ["web", i] do |web|
      web.vm.box = $box_name
      web.vm.box_url = $box_url
      ip = "192.168.50.#{i+100}"
      web.vm.network "private_network", ip: ip

      web.vm.provision "shell", inline: <<-SHELL
        systemctl disable firewalld.service
        systemctl stop firewalld.service
        yum -y install httpd
        systemctl enable httpd.service
        systemctl start httpd.service
        echo "<h1>#{i+100}</h1>" > /var/www/html/index.html
      SHELL
    end
  end

  config.vm.define "haproxy" do |haproxy|
    haproxy.vm.box = $box_name
    haproxy.vm.box_url = $box_url
    haproxy.vm.network "private_network", ip: "192.168.50.10"

    haproxy.vm.provision "shell", inline: <<-SHELL
      systemctl disable firewalld.service
      systemctl stop firewalld.service
      yum -y install haproxy
      mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.org
      systemctl enable haproxy.service
    SHELL

    haproxy.vm.provision "file", source: "haproxy.cfg", destination: "~/haproxy.cfg"

    (1..$num_instances).each do |i|
      ip = "192.168.50.#{i+100}"
      haproxy.vm.provision "shell", inline: <<-SHELL
        echo "    server  app#{i} #{ip}:80 check" >> /home/vagrant/haproxy.cfg
      SHELL
    end

    haproxy.vm.provision "shell", inline: <<-SHELL
      mv /home/vagrant/haproxy.cfg /etc/haproxy/
      systemctl start haproxy.service
    SHELL
  end

end
