Muodosta haluttuun kansiorakenteeseen polku, tämä onnistuu käyttäen Windowsin graafista käyttöliittymää tai Admin Powerhellissä.

    mkdir C:/Users/käyttäjä/Kohde_kansio

Kun haluttu kohde kansio on olemassa, voidaan siihen siirtyä Admin Powershellissä.

    Set-Location C:/Users/käyttäjä/Kohde_kansio

Admin Powershellin sijainnin ollessa halutussa kohde kansiossa, voidaan alustaa Vagrant

    vagrant init debian/bullseye64

Kohde kansioon muodostuvaa Vagratfileä voidaan muokata joko Admin Powershellistä tai graafisen käyttöliittymän kautta valitussa editorissa. Tässä graafisen käyttöliittymän kautta valittu editori on nopeampi.
Käytetään Tero Karvisen muodostamaa esimerkkiä uuden Vagrantfilen pohjana. Tämä rakenne muodostaa kolme konetta, yhden isännän (tmaster) ja kaksi alaista (t001 sekä t002).
Muodostettavat koneet ajetaan suoraan isäntä/alainen rakenteellisiksi. Manuaalinen esimerkki tämän toteuttamisesta löytyy repositiosta: tähän sen repo esimerkki

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

Tämän jälkeen voidaan Admin Powershellillä alustaa Vagrantfilellä muodostetut käyttäjät VM:ään aktiivisiksi, uusiksi koneiksi. Tässä saattaa mennä useitakin minuutteja, varsinkin jos koneita on monta.

    vagrant up

Kun uudet koneet ovat muodostuneet, voidaan niihin kirjautua. 
Esimerkkinä käytetty koodi muodostaa koneet nimiltä: t001, t002 ja tmaster.

    vagrant ssh halutun_koneen_nimi

Muodostetussa Vagrantfilessä isäntä/alainen suhde on muodostettu osana toteutusta, joten tässä tapauksessa voimme kirjautua sisään suoraan isäntään

    vagrant ssh tmaster

Tämän jälkeen voimme hyväksyä odottavat alaiset. Muista vahvistaa listatut koneet lisäyksessä syöttämällä y + enter.

    sudo salt-key -A

Muodostuksen onnistumista voi testata millä tahansa komennolla, jossa kutsutaan alaisia.

    sudo salt '*' cmd.run 'whoami'
    sudo salt '*' test.ping

Jos alaiset vastaavat, voidaan yhteyttä käyttä esim. tietojen keräämiseen. Pelkkä grains.items antaa erittäin paljon tietoa, joten hakua on hyvä rajata.

    sudo salt '*' grains.items
    sudo salt '*' grains.item osfinger ipv4

Kun isäntä/alainen yhteys on todettu toimivaksi, voidaan aloittaa idempotenttiin johtavien toimien tekeminen, joka tarkoittaa muutoksien tekemistä ja niiden toteutumisen varmistamista (Eli komennon toiseen kertaan totettamista). 
Tämä voidaan toteuttaa vaikka asentamalla alaisille ohjelmia... 

    sudo salt '*' state.single pkg.installed ohjelman_nimi
    sudo salt '*' state.single pkg.installed apache2
    sudo salt '*' state.single pkg.installed micro


...Tarkistamalla asennettujen ohjelmien tila...

    sudo salt '*' state.single service.running ohjelman_nimi
    sudo salt '*' state.single service.running apache2

...Tai keskeyttämällä ohjelman toiminta etänä.

    sudo salt '*' state.single service.dead ohjelman_nimi
    sudo salt '*' state.single service.dead apache2
