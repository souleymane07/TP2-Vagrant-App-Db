Vagrant.configure("2") do |config|
  # Configuration de base pour les 2 machines
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_check_update = false

  # Définition de la première machine : srv-app
  config.vm.define "srv-app" do |app|
    app.vm.hostname = "srv-app"
    app.vm.network "private_network", ip: "192.168.33.10"
    app.vm.network "forwarded_port", guest: 8080, host: 8080  # Pour Tomcat
    
    app.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = "2"
      vb.name = "srv-app"
    end
  end

  # Définition de la deuxième machine : srv-db
  config.vm.define "srv-db" do |db|
    db.vm.hostname = "srv-db"
    db.vm.network "private_network", ip: "192.168.33.11"
    # On change le port pour éviter le conflit : 3307 au lieu de 3306
    db.vm.network "forwarded_port", guest: 3306, host: 3307
    
    db.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "1024"
      vb.cpus = "1"
      vb.name = "srv-db"
    end
  end
end
