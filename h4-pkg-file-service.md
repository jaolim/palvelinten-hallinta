# h4 Pkg-file-service

## Ympäristö

### Rauta

- **Käyttöjärjestelmä:** Windows 11 Education 23H2 64-bit

- **CPU:** AMD Ryzen 9 3900XT 12-Core 3.80GHz

- **RAM:** 32GB

- **Kiintolevy:** 2TB Samsung M.2 SSD

- **GPU:** Nvidia GeForce RTX 3080

### Virtuaali

- **Virtualisointi:** Oracle VirtualBox 7.1.6

- **Automatisointi:** Vagrant 2.4.3

- **Käyttöjärjestelmä:** Debian 12 6mo4-bit

### Konfiguraatio

*Luo yhden orjan ja yhden mestarin ympäristön, jossa virtuaalikoneiden välinen kommunikaatio tapahtuu yksityisessä virtuaaliverkossa.*
*Välillä kommunikaatio ei toimi suoraan, mutta tämän saa korjattua ajamalla* ```vagrant reload``` *komennon.*


**Vagrantfile:**

```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"


  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.2.10"
    master.vm.provision "shell", path: "provision/master.sh"
  end

  config.vm.define "slave001" do |slave001|
    slave001.vm.hostname = "slave001"
    slave001.vm.network "private_network", ip: "192.168.2.11"
    slave001.vm.provision "shell", path: "provision/slave.sh"
  end
  
end

```

**master.sh:**

```
sudo apt-get update
sudo apt-get install curl -y
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get install salt-master -y
```

**slave.sh:**

```
sudo apt-get update
sudo apt-get install curl -y
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get install salt-minion -y
sudo echo "master: 192.168.2.10" | sudo tee -a /etc/salt/minion
sudo systemctl restart salt-minion
```

## x) Lue ja tiivistä

### Karvinen 2018: [Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port](https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh)

Linuxissa demonien asetukset konfiguraatiotiedostoissa, joten niiden hallintaa haluaa automatisoida, tarvitaan master koneelle pohjatiedosto, jonka muuttuessa myös orjakoneiden tiedostoja muutetaan.

SSHd:n kohdalla tämä tiedosto on ```/etc/ssh/sshd_config```, ja jotta demoni tiedetään käynnistää uudelleen tiedoston muuttuessa, pitää salt komentaa myös tarkkailemaan tätä tiedostoa.

/srv/salt/sshd.sls:
```
openssh-server:
 pkg.installed
/etc/ssh/sshd_config:
 file.managed:
   - source: salt://sshd_config
sshd:
 service.running:
   - watch:
     - file: /etc/ssh/sshd_config
```
*openssh-server paketin tulee olla asennettuna*

*/etc/ssh/sshd_config:n sisällön tulee olla salt://sshd_config lähteen mukainen*

*sshd:n tulee uudelleenkäynnistyä, kun /etc/ssh/sshd_config tiedostoa on muokattu*

## a) Apache easy mode.

*Asenna Apache, korvaa sen testisivu ja varmista, että demoni käynnistyy.*

Loin yhden mestarin ja yhden orjan ympäristön ajamalla ```vagrant up``` ja ```vagrant reload``` komennot käyttäen *konfiguraatio* kohdassa listattuja asetuksia.

Olin jo tehtävässä [h3 - Infraa Koodina](https://github.com/jaolim/palvelinten-hallinta/blob/main/h3-infraa-koodina.md) tehnyt Apachen etusivun korvaavan tilan, joten päätin kirjoittaa automatisoinnin suoraan, koska tiesin jo korvattavan sivun olevan polussa ```/var/www/html/index.html```.

Hyväksyin orja-avaimen ja kirjoitin ensin version, joka varmistaa apachen olevan asennettuna, pyörimissä, ja etusivun tiedoston olemassa.

/srv/salt/apache/init.sls:
```
apache2:
  pkg.installed: []
  service.running:
    - require:
      - pkg: apache2
/var/www/html/index.html:
  file.managed
```
![Apache installed](/h4/h4_a01.png)

Tämän jälkeen lisäsin sivun sisällön samaan kansioon *init.sls:n* kanssa ja määrittelin tilan käyttämään tätä lähteenä etusivun sisällölle.

/srv/salt/apache/init.sls:

```
apache2:
  pkg.installed: []
  service.running:
    - require:
      - pkg: apache2
/var/www/html/index.html:
  file.managed:
    - source: salt://apache/index.html
```

/srv/salt/apache/index.html:

```
Edits will be overwritten, make a new virtual host instead.
```

Ajoin *apache* tilan ja salt väitti ylikirjoituksen onnistuneen.

![Overwrite index.html](/h4/h4_a02.png)

Varmistin vielä tämän käskemällä orjaa curlaamaan localhostin:

```sudo salt '*' state.single cmd.run 'curl localhost'```

![curl response](/h4/h4_a03.png)

**Ajankäyttö:** 9 minuuttia. Vagrantin käyttämä aika ympäristön luontiin jätetty pois laskuista.

## b) SSHouto.

*Lisää uusi portti, jossa SSHd kuuntelee.*

Asensin ensin *openssh-serverin* ```sudo apt-get install openssh-server``` ja varmistin *sshd_config* tiedoston löytyvän ```cat /etc/ssh/sshd_config```.

Tämän jälkeen loin *sshd* tilaa varten kansion, ja kopioin *sshd_configin* tänne ```sudo cp /etc/ssh/sshd_config /srv/salt/sshd/sshd_config```.

Lisäsin *ssh_config* tiedostoon portit, jotka aion avata:

```
port 22
port 55555
```

Varmistin, tämän jälkeen, ettei orja jo kuuntele määriteltyä porttia.

![Connection refused](/h4/h4_b01.png)

*Oletusporttiin kohdistunut pyyntö meni läpi ja hylättiin väärään avaimen takia, ja portti 55555 hylättiin portin takia.*

Määrittelin nyt tilan ja ajoin sen ```sudo salt '*' state.apply sshd```.

/var/salt/sshd/init.sls:

```
openssh-server:
  pkg.installed
/etc/ssh/sshd_config:
  file.managed:
    - source: salt://sshd/sshd_config
sshd:
  service.running:
    - watch:
      - file: /etc/ssh/sshd_config
```

![It listens](/h4/h4_b02.png)

*Tila ajettiin onnistuneesti ja ssh yhteydenottoyritys varmisti, että porttia kuunneltiin.*

**Ajankäyttö: ** 24 minuuttia.

## c) Vapaaehtoinen, haastavahko tässä vaiheessa: Asenna ja konfiguroi Apache ja Name Based Virtual Host.

*Sen tulee näyttää palvelimen etusivulla weppisivua. Weppisivun tulee olla muokattavissa käyttäjän oikeuksin, ilman sudoa.*

Lähdin ensin luomaan weppisivun sisällön ja virtual hostin *master* koneella manuaalisesti.

Asensin micron ja tein sillä index.html sivun kansioon ```~/public/html/default.site.com/index.html```.

index.html:

```
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8"/>
		<title>Home</title>
	</head>
	<body>
		<h1>Hello localhost!</h1>
	</body>
</html>
```

Tämän jälkeen lähdin määrittämään virtualhostin. Muistin virkistykseksi kaivoin tähän Linux-palvelimet kurssilta [tutut ohjeet asiasta](https://terokarvinen.com/2018/04/10/name-based-virtual-hosts-on-apache-multiple-websites-to-single-ip-address/).

default.site.com.conf:

```
<VirtualHost *:80>
  ServerName default.site.com
  ServerAlias www.default.site.com localhost http://localhost
  DocumentRoot /home/vagrant/public/html/default.site.com
  <Directory /home/vagrant/public/html/default.site.com>
    Require all granted
  </Directory>
</VirtualHost>
```

![Manual works](/h4/h4_c01.png)

Varmistettuani konfiguraation toimivuuden lähdin automatisoimaan sitä saltilla.

Loin kansion tilaa *apache-default* varten ja kopioin aiemmat *index.html* ja *default.site.com.conf* tiedostot pohjiksi.

```sudo cp /home/vagrant/public/html/default.site.com/index.html /srv/salt/apache-default/index.html```

```sudo cp /etc/apache2/sites-available/default.site.com.conf /srv/salt/apache-default/default.site.com.conf```

Kirjoitin ensin version, joka tarkistaa apachen ja tiedostojen oikeat tilat, muttei vielä aktivoi sivua.

/srv/salt/apache-default/init.sls:

```
apache2:
  pkg.installed: []
  service.running:
    - require:
      - pkg: apache2
/home/vagrant/public/html/default.site.com/index.html:
  file.managed:
    - source: salt://apache-default/index.html
/etc/apache2/sites-available/default.site.com.conf:
  file.managed:
    - source: salt://apache-default/index.html
```

![Missing parent](/h4/h4_c02.png)

*Tila epäonnistui, koska tilaa ei oltu määritelty luomaan parent kansiota tarvittaessa.*

Kaivoin [saltin dokumenaatiosta *makedirs* komennon](https://docs.saltproject.io/en/3006/ref/states/all/salt.states.file.html) tätä varten ja päivitin init.sls:n muotoon:

```
apache2:
  pkg.installed: []
  service.running:
    - require:
      - pkg: apache2
/home/vagrant/public/html/default.site.com/index.html:
  file.managed:
    - makedirs: True
    - source: salt://apache-default/index.html
/etc/apache2/sites-available/default.site.com.conf:
  file.managed:
    - source: salt://apache-default/index.html
```

Tämän jälkeen myös index.html luonti onnistui.

Tässä kohtaan tuli pienehkö ongelma vastaan, koska manuaalisen tavan replikointi olisi vaatinut ```a2ensite``` komennon ajoa, jonka idempotentti toteutus olisi ollut työlästä.

Kaivoin tähän [Karvisen ohjeet vuodelta 2018](https://terokarvinen.com/2018/apache-user-homepages-automatically-salt-package-file-service-example/?fromSearch=apache%20salt), joista opin, että konfiguraatio tiedostosta symlinkin luontiin löytyy saltista työkalu.

init.sls:
```
apache2:
  pkg.installed: []
  service.running:
    - require:
      - pkg: apache2
	- watch:
	  - file: /etc/apache2/sites-enabled/default.site.com.conf
/home/vagrant/public/html/default.site.com/index.html:
  file.managed:
    - makedirs: True
    - source: salt://apache-default/index.html
/etc/apache2/sites-available/default.site.com.conf:
  file.managed:
    - source: salt://apache-default/index.html
/etc/apache2/sites-enabled/default.site.com.conf:
  file.symlink:
    - target: ../sites-available/default.site.com.conf
```

Koitin ajaa tilan, mutta tämä päättyi service.running kohdalta virheeseen.

![Partial success](/h4/h4_c03.png)

*apache2 epäonnistui, mutta muut menivät läpi.*

Katsoin vielä ohjeita tarkemmin, ja siirsin watchin omaan osioonsa *apache2service* alle.

init.sls:

```
apache2:
  pkg.installed: []
  service.running:
    - require:
      - pkg: apache2
/home/vagrant/public/html/default.site.com/index.html:
  file.managed:
    - makedirs: True
    - source: salt://apache-default/index.html
/etc/apache2/sites-available/default.site.com.conf:
  file.managed:
    - source: salt://apache-default/index.html
/etc/apache2/sites-enabled/default.site.com.conf:
  file.symlink:
    - target: ../sites-available/default.site.com.conf
apache2service:
  service.running:
    - name: apache2
    - watch:
      - file: /etc/apache2/sites-enabled/default.site.com.conf
```

Tälläkin tavalla toteutettu versio aiheutti virhettä, joten päätin ottaa orjakoneeseen ssh yhteyden ja tutkia, olivatko tiedostot oikein.

Huomasin VirtualHost tiedoston sisältävän index.html:n sisällön, joten korjasin init.sls:ssä virtual hostin polun määrityksen.

init.sls:
```
apache2:
  pkg.installed
/home/vagrant/public/html/default.site.com/index.html:
  file.managed:
    - makedirs: True
    - source: salt://apache-default/index.html
/etc/apache2/sites-available/default.site.com.conf:
  file.managed:
    - source: salt://apache-default/default.site.com.conf
/etc/apache2/sites-enabled/default.site.com.conf:
  file.symlink:
    - target: ../sites-available/default.site.com.conf
apache2service:
  service.running:
    - name: apache2
    - watch:
      - file: /etc/apache2/sites-enabled/default.site.com.conf
```

Tämä aiheutti edelleen virheen, joten päätin lähteä orjakoneelle manuaalisesti selvittämään ongelmaa.

```sudo systemctl restart apache2``` aiheutti virhettä vaikka ajoin ```sudo apt-get purge apache2``` ja ```sudo apt-get install apache2```.

Päätin tämän jälkeen asentaa *salt-minionin* master koneeseen ja ajaa tilan lokaalisti.

![Works locally](/h4/h4_c04.png)

*Komento ei sentään riko jo valmiiksi toimivaa tilaa!*

Tuhosin koneen *slave001* ja uudelleenasensin sen oletusasetuksilla ```vagrant destroy slave001``` & ```vagrant up```.

Koneiden välinen yhteys ei toiminut luotettavasti, joten tuhosin koko ympäristön ja loin sen uudelleen.

Hyväksyttyäni orjakoneen avaimen ja luotuani tiedostot uudestaan, kokeilin ajaa tilaa orjakoneelle.

Nyt komento toimi onnistuneesti.

![Success1](/h4/h4_c05.png)

![Success2](/h4/h4_c06.png)

Varmistin vielä sivun muuttuneen ajamalla ```sudo salt '*' state.single cmd.run 'curl localhost'```.

![Confirmed to work](/h4/h4_c07.png)

**Ajankäyttö:** 2 tuntia 13 minuuttia.

## Lähteet

#### Tehtävä

Karvinen, T. 2025. [Pkg-file-service](https://terokarvinen.com/palvelinten-hallinta/#h4-pkg-file-service)

#### Lyhennettävät

Karvinen 2018: [Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port](https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh)

#### Tiedonhaku

Karvinen 2018: [Apache User Homapages Automatically - Salt Package-File-Service Example](https://terokarvinen.com/2018/apache-user-homepages-automatically-salt-package-file-service-example/?fromSearch=apache%20salt)

Karvinen 2018: [Name Based Virtual Hosts on Apache - Multiple Websites to Single IP Address](https://terokarvinen.com/2018/04/10/name-based-virtual-hosts-on-apache-multiple-websites-to-single-ip-address/)

Saltproject.io. [salt.states.file](https://docs.saltproject.io/en/3006/ref/states/all/salt.states.file.html)

### Lisenssi

Sivun sisältöä saa levittää GPL-3.0 lisenssin sallimin ehdoin: https://github.com/jaolim/palvelinten-hallinta/tree/main?tab=GPL-3.0-1-ov-file