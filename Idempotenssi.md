Muodosta haluttuun kansiorakenteeseen polku, tämä onnistuu käyttäen Windowsin graafista käyttöliittymää tai Admin Powerhellissä.

    mkdir C:/Users/käyttäjä/Kohde_kansio

Kun haluttu kohde kansio on olemassa, voidaan siihen siirtyä Admin Powershellissä.

    Set-Location C:/Users/käyttäjä/Kohde_kansio

Admin Powershellin sijainnin ollessa halutussa kohde kansiossa, voidaan alustaa Vagrant

    vagrant init debian/bullseye64

Muodostuvaa Vagratfileä voidaan muokata joko Admin Powershellistä tai graafisen käyttöliittymän kautta valitussa editorissa. Tässä graafisen käyttöliittymän kautta valittu editori on nopeampi.
Käytetään Tero Karvisen muodostamaa esimerkkiä uuden Vagrantfilen pohjana.

    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    # Copyright 2014-2023 Tero Karvinen http://TeroKarvinen.com

    $minion = <<MINION
    sudo apt-get update
    sudo apt-get -qy install salt-minion
    echo "master: 192.168.12.3">/etc/salt/minion
    sudo service salt-minion restart
    echo "See also: https://terokarvinen.com/2023/salt-vagrant/"
    MINION

    $master = <<MASTER
    sudo apt-get update
    sudo apt-get -qy install salt-master
    echo "See also: https://terokarvinen.com/2023/salt-vagrant/"
    MASTER

    Vagrant.configure("2") do |config|
	    config.vm.box = "debian/bullseye64"

	    config.vm.define "t001" do |t001|
		    t001.vm.provision :shell, inline: $minion
		    t001.vm.network "private_network", ip: "192.168.12.100"
		    t001.vm.hostname = "t001"
	    end

	    config.vm.define "t002" do |t002|
		    t002.vm.provision :shell, inline: $minion
		    t002.vm.network "private_network", ip: "192.168.12.102"
		    t002.vm.hostname = "t002"
	    end

	    config.vm.define "tmaster", primary: true do |tmaster|
		    tmaster.vm.provision :shell, inline: $master
		    tmaster.vm.network "private_network", ip: "192.168.12.3"
		    tmaster.vm.hostname = "tmaster"
	    end
end
