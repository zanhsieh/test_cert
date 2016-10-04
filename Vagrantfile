# -*- mode: ruby -*-
# vi: set ft=ruby :

$IPs = {
  "m1" => "10.0.40.41",
}

def host_check(ips={})
  return ips.map {|k,v|<<-INNER
  if [ ! `grep -q #{v} /etc/hosts` ]; then
    echo '#{v} #{k}' | sudo tee -a /etc/hosts
  fi
  INNER
  }.join()
end

Vagrant.configure("2") do |config|
  config.vm.box = "debian/contrib-jessie64"
  config.vm.box_check_update = false
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = "box"
    config.cache.synced_folder_opts = {
      type: "nfs",
      mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    }
  end
  $IPs.map do |k,v|
    config.vm.define "#{k}" do |m|
      m.vm.hostname = "#{k}"
      m.vm.network "private_network", ip: "#{v}"
      m.vm.provision "shell", inline:<<-SHELL
      #{host_check($IPs)}
      SHELL
      m.vm.provision "shell", inline:<<-SHELL
      sudo apt update
      sudo apt -y upgrade
      sudo apt -y install apache2 libapache2-mod-proxy-html rsync curl
      sudo rsync -avz /vagrant/etc/ssl/ /etc/ssl/
      sudo rsync -avz /vagrant/etc/apache2/ /etc/apache2/
      sudo rsync -avz /vagrant/var/ /var/
      sudo service apache2  restart
      echo '#{$IPs[k]} www.example.com' | sudo tee -a /etc/hosts
      echo '#{$IPs[k]} www.sample.com' | sudo tee -a /etc/hosts
      SHELL
    end
  end
end
