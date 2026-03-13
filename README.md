# TP2-Vagrant-App-Db

## Objectif du TP
Créer deux machines virtuelles avec Vagrant :
- **srv-app** : Serveur d'application avec Tomcat et une application web Java
- **srv-db** : Serveur de base de données avec MySQL

## Prérequis

- **Vagrant** installé sur la machine hôte
- **VirtualBox** installé
- **Git** pour la gestion de version
- Un terminal

## Partie 1 : Création des VM avec Vagrant

### 1.1 Configuration du Vagrantfile

Le fichier Vagrantfile définit deux machines avec leurs caractéristiques :


Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_check_update = false

  # Machine srv-app
  config.vm.define "srv-app" do |app|
    app.vm.hostname = "srv-app"
    app.vm.network "private_network", ip: "192.168.33.10"
    app.vm.network "forwarded_port", guest: 8080, host: 8080
    app.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = "2"
      vb.name = "srv-app"
    end
  end

  # Machine srv-db
  config.vm.define "srv-db" do |db|
    db.vm.hostname = "srv-db"
    db.vm.network "private_network", ip: "192.168.33.11"
    db.vm.network "forwarded_port", guest: 3306, host: 3307  # MySQL sur port 3307
    db.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "1024"
      vb.cpus = "1"
      vb.name = "srv-db"
    end
  end
end

## Partie 2 : Lancement des machines
vagrant up

<img width="1459" height="703" alt="image" src="https://github.com/user-attachments/assets/9c994582-e679-4fee-bc28-4c6d89c77475" />
