# vagrant
Vagrantfile
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

  NodeCount = 1

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
end
