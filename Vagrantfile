Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.hostname = "alejandro.test"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
    vb.name = "nginx"
  end

  # Solo instalación básica, sin configuración de Nginx
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y
    echo "Máquina base creada. Instala y configura Nginx manualmente."
  SHELL
end