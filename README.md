**Развернуть несколько виртуальных машин  VirtualBox**

1. подготовка 

Генерируем ключ ssh

`ssh-keygen -t ed25519 -N "new_passphrase" -f ./id_ed25519`

2. Для запуска vagrant создаем файл "Vagrantfile"
<details>
  <summary>Содержимое Vagrantfile</summary>

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_SERVER_URL'] = 'http://vagrant.elab.pro'
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

Vagrant.configure("2") do |config|

  config.vm.define "mydebian" do |kmaster|
    kmaster.vm.box = "debian/bullseye64"
    kmaster.vm.hostname = "kmaster.local"
    kmaster.vm.network "private_network", ip: "192.168.56.10/24"
    kmaster.vm.provider "virtualbox" do |v|
      v.name =  "mydebian"
      v.memory = 2048
      v.cpus = 2
    end
  end
  
  NodeCount = 0

  (1..NodeCount).each do |i|
    config.vm.define "kworker#{i}" do |workernode|
      workernode.vm.box = "debian/bullseye64"
      workernode.vm.hostname = "kworker#{i}.local"
      workernode.vm.network "private_network", ip: "192.168.56.2#{i}"
      workernode.vm.provider "virtualbox" do |v|
        v.name = "kworker#{i}"
        v.memory = 1024
        v.cpus = 1
      end
    end
  end 
  
config.vm.provision "shell" do |s|
  ssh_prv_key = ""
  ssh_pub_key = ""
  if File.file?("./id_ed25519")
    ssh_prv_key = File.read("./id_ed25519")
    ssh_pub_key = File.readlines("./id_ed25519.pub").first.strip
  else
    puts "No SSH key found. You will need to remedy this before pushing to the repository."
  end
  s.inline = <<-SHELL
    if grep -sq "#{ssh_pub_key}" /home/vagrant/.ssh/authorized_keys; then
      echo "SSH keys already provisioned."
      exit 0;
    fi
    echo "SSH key provisioning."
    mkdir -p /home/vagrant/.ssh/
    touch /home/vagrant/.ssh/authorized_keys
    echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
    echo #{ssh_pub_key} > /home/vagrant/.ssh/id_rsa.pub
    chmod 644 /home/vagrant/.ssh/id_rsa.pub
    echo "#{ssh_prv_key}" > /home/vagrant/.ssh/id_rsa
    chmod 600 /home/vagrant/.ssh/id_rsa
    chown -R vagrant:vagrant /home/vagrant
    exit 0
  SHELL
end

end
```
</details>


