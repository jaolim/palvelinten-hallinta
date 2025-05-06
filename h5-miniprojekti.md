# h5-miniprojekti

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

- **Käyttöjärjestelmä:** Debian 12 64-bit

## a) Oma miniprojekti - [Salt admin setup](https://github.com/jaolim/salt-admin-setup)

*Projektirepo löytyy otsikon linkistä, tämä sivu on raportti projektin etenemisestä. Proktisivun README.md on parempi lähde lopputuloksen kuvaukseen.*

Lähtösuunnitelmani on toteuttaa tila kaikille orjakoneille, joka varmistaa admin ryhmään kuuluvan sudo käyttäjän olemassaolon ja ssh kirjautumisen tälle käyttäjälle jokaiselle orjakoneella. Muut tilat tulevat menemään koneen tyypin mukaan määriteltynä koneen nimellä Top filessä.

Ympäristöön käytän vagranttia ja kahta *vagrant upin* yhteydessä ajattavaa skriptiä saltin asentamiseksi master ja orjakoneille.

En kirjoittanut raporttia suoraan tehdässä, vaan ainoastaan jokaisen onnistuneen vaiheen jälkeen, joten tämä raportti sisältää vain toimivat kokonaisuudet vaiheittain ja lopussa viimeisimmän version päivitettynä.
Koodin etenemistä voi seurata projektirepon commit historiasta.

Vaiheista riippumatta asennusprosessi käyttämässäni vagrant ympäristössä on sama:

```
vagrant up
vagrant reload # ilman tätä koneiden välillä ilmenee usein yhteysongelmia omalla koneellani, mahdollisesti turha muilla koneilla
vagrant ssh master
sudo salt-key -A -y
git clone https://github.com/jaolim/salt-admin-setup.git
cd salt-admin-setup
bash master-module.sh
sudo salt '*' state.apply
```

### Vaihe 1: vagrant ympäristö, ssh tunnistautuminen ja control käyttäjä

**Vagrantfile:**

```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"


  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.2.10"
	master.vm.provision "shell", path: "provision/master.sh"
  end

  config.vm.define "webminion01" do |webminion01|
    webminion01.vm.hostname = "webminion01"
    webminion01.vm.network "private_network", ip: "192.168.2.11"
	webminion01.vm.provision "shell", path: "provision/slave.sh"
  end
  
#  config.vm.define "minion01" do |minion01|
#    minion01.vm.hostname = "minion01"
#    minion01.vm.network "private_network", ip: "192.168.2.12"
#    minion01.vm.provision "shell", path: "provision/slave.sh"
#  end
  
end
```
- kahden koneen salt ympäristö, joka käyttää yksityistä virtuaaliverkkoa keskinäiseen kommunikointiin
- kolmannen koneen saa poistamalla kommentit

**master.sh:**

```
sudo apt-get update
sudo apt-get install curl -y
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get install salt-master -y
sudo apt-get install git -y
```
- asentaa salt-masterin
- asentaa gitin

**slave.sh:**

```
sudo apt-get update
sudo apt-get install curl -y
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get install salt-minion -y
echo "master: 192.168.2.10" | sudo tee -a /etc/salt/minion #update master IP
sudo systemctl restart salt-minion
```

- asentaa curlin
- asentaa salt-minionin
- uudelleenkäynnistää salt-minionin

**master-module.sh:**

```
ssh-keygen -q -t rsa -N "" -f ../.ssh/id_rsa <<< y
sudo mkdir /srv/salt
sudo cp -a ./admin /srv/salt/admin
sudo cp -a ./web /srv/salt/web
sudo cp -a ../.ssh/id_rsa.pub /srv/salt/admin/id_rsa.pub
sudo cp -a ./top.sls /srv/salt/top.sls
```

- generoi ssh avaimanen
- tekee saltille kansion
- kopioi ssh avaimen ja admin sekä web tilat

**/srv/salt/admin/init.sls:**

```
admin:
  group.present:
    - gid: 1234

control:
  user.present:
    - fullname: Boss
    - shell: /bin/bash
    - password: '$6$FMzzQ..34PLTgqMd$FqXe3tmhA6VbNmgNW7dziCraT5BjyBVMnK8wYPquh9H9zcETWMYSZYU89BFut4QQomBQ6UDtP5nNvqhGElFdd.'
    - home: /home/control
    - uid: 1234
    - gid: 1234
    - groups:
      - sudo
      - admin

sshkey:
  ssh_auth:
    - present
    - require:
      - user: control
    - user: control
    - source: salt://admin/id_rsa.pub
```

- admin ryhmä on olemassa
- control käyttäjä on olemassa
 - control kuuluu ryhmään admin ja sudo
 - controllilla on tietty salasana (uuden hashatyn salasanan voi generoida lokaalisti ajamalla ```sudo salt-call --local shadow.gen_password 'your password'```)
- ssh tunnistus on olemassa käyttäen lähdetiedoston avainta

### Vaihe 2: Ajettavat moduulit minionin nimen mukaan ja web moduuli

**top.sls**

```
base:
  '*':
    - admin
  'web*':
    - web
```

- admin tila ajatean kaikille koneille
- web tila ajetaa web-alkuisille koneille

Web tilaan käytin pohjana tämän kurssin tehtävän [h4 Pkg-file-service ratkaisuani](https://github.com/jaolim/palvelinten-hallinta/blob/main/h4-pkg-file-service.md), jota laajensin käsittämään oikeuksien määrittelyn.

**/srv/salt/web/init.sls:**

```
apache2:
  pkg.installed
/home/control/public/html/default.com/index.html:
  file.managed:
    - makedirs: True
    - user: control
    - group: admin
    - source: salt://web/index.html
/etc/apache2/sites-available/default.com.conf:
  file.managed:
    - source: salt://web/default.com.conf
/etc/apache2/sites-enabled/default.com.conf:
  file.symlink:
    - target: ../sites-available/default.com.conf
apache2service:
  service.running:
    - name: apache2
    - watch:
      - file: /etc/apache2/sites-enabled/default.com.conf
```

- apache2 on asennettuna
- virtual host default.com on oikeassa tilassa
- webbisivun sisältö on oikein

**/srv/salt/web/default.com.conf:**

```
<VirtualHost *:80>
  ServerName default.com
  ServerAlias www.default.com localhost http://localhost
  DocumentRoot /home/control/public/html/default.com
  <Directory /home/control/public/html/default.com>
    Require all granted
  </Directory>
</VirtualHost>
```

- Virtual hostin lähdetiedosto

**/srv/salt/web/index.html:**

```
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8"/>
		<title>Home</title>
	</head>
	<body>
		<h1>Hello web minion!</h1>
	</body>
</html>
```

- Webbisivun lähdetiedosto

### Vaihe: 3 UFW kaikille ja custom portit webille

Lisäsin admin tilaan UFW:tä koskevat rivit.

/srv/salt/admin/init.sls:

```
...
ufw:
  pkg.installed

ufw_service:
  service.running:
    - name: ufw

ufw enable:
  cmd.run:
    - unless: "ufw status | grep 'Status: active'"

ufw allow 22/tcp:
  cmd.run:
  - unless: "ufw status | grep '22/tcp'"
```

- Koska halusin kaikille koneille portin 22/tcp:n auki, mutta web koneille muitakin portteja, tein määrittelyn ajattavalla komennolla lähdetiedoston sijaan.
- Tilasta on tehty idempotentti hakemalla ufw status kyselystä termiä '22/tcp'.

Päivitin myös web tilaa omilla porttimäärityksillä ufw:lle:

```
...
ufw allow 80/tcp:
  cmd.run:
    - unless: "ufw status | grep '80/tcp'"

ufw allow 443/tcp:
  cmd.run:
    - unless: "ufw status | grep '443/tcp'"
```

### Vaihe 4: Apachen oletussivun poisto

Olin jo web tilaa työstäessä kokeillut oletussivun poistoa, mutta ongelma tuli vastaan apachen ehdollisessa uudelleenkäynnistämisessä, joten jätin sen myöhemmälle.

Päädyin ratkaisemaan tämän laittamalla *a2dissite* ja *restart* komennat ajettaksi peräkkäin, ja tämän ajettavaksi vain jos *000-default.conf* löytyy */etc/apache2/sites-enabled* kansiosta.

```
...
a2dissite 000-default.conf && systemctl restart apache2:
  cmd.run:
    - onlyif: "ls /etc/apache2/sites-enabled | grep '000-default.conf'"
...
```

### Lopullinen versio:

*Listaan admin ja web init.sls tiedostot, muita tiedostoja ei ole muokattu edellisestä.*

/srv/salt/admin/init.sls:

```
apache2:
  pkg.installed

/home/control/public/html/default.com/index.html:
  file.managed:
    - makedirs: True
    - user: control
    - group: admin
    - source: salt://web/index.html

/etc/apache2/sites-available/default.com.conf:
  file.managed:
    - source: salt://web/default.com.conf

/etc/apache2/sites-enabled/default.com.conf:
  file.symlink:
    - target: ../sites-available/default.com.conf

apache2service:
  service.running:
    - name: apache2
    - watch:
      - file: /etc/apache2/sites-enabled/default.com.conf

disable-default:
  cmd.run:
    - names:
      - a2dissite 000-default.conf
      - systemctl restart apache2
    - onlyif: "ls /etc/apache2/sites-enabled | grep '000-default.conf'"

ufw allow 80/tcp:
  cmd.run:
    - unless: "ufw status | grep '80/tcp'"

ufw allow 443/tcp:
  cmd.run:
    - unless: "ufw status | grep '443/tcp'"
```

/srv/salt/web/init.sls:

```
apache2:
  pkg.installed

/home/control/public/html/default.com/index.html:
  file.managed:
    - makedirs: True
    - user: control
    - group: admin
    - source: salt://web/index.html

/etc/apache2/sites-available/default.com.conf:
  file.managed:
    - source: salt://web/default.com.conf

/etc/apache2/sites-enabled/default.com.conf:
  file.symlink:
    - target: ../sites-available/default.com.conf

apache2service:
  service.running:
    - name: apache2
    - watch:
      - file: /etc/apache2/sites-enabled/default.com.conf

a2dissite 000-default.conf && systemctl restart apache2:
  cmd.run:
    - onlyif: "ls /etc/apache2/sites-enabled | grep '000-default.conf'"

ufw allow 80/tcp:
  cmd.run:
    - unless: "ufw status | grep '80/tcp'"

ufw allow 443/tcp:
  cmd.run:
    - unless: "ufw status | grep '443/tcp'"
```

### Lopputestaus

Käytin kolmen koneen vagrant asetusta (kolmannesta koneesta poistettu kommentit vagrantfilestä).

Ennen kuvakaappauksia ajoin komennot ```vagrant up``` ja ```vagrant reload```.

**Komennot:**
![Commands](/h5/h5_a01.png)

- ```ls /srv/salt``` - kansiota ei ole
- ```sudo salt-key -A -y``` - hyväksytään kahden orjakoneen avaimet
- ```curl 192.168.2.11``` - orjakone webminion01 ei tarjoile weppisivua
- ```git clone https://github.com/jaolim/salt-admin-setup.git``` - kloonataan projektirepo
- ```cd salt-admin-setup``` - siirrytään projektirepoon
- ```bash master-module.sh``` - ajetaan ssh avaimen luonti/kopiointi ja moduulien asennus
- ```ls /srv/salt``` - varmistetaan moduulien nyt löytyvän
- ```sudo salt '*' state.apply``` - ajetaan moduulit

**Vastaukset:**

*Otin kuvakaappaukset kahdesta ajokerrasta idempotenssin näyttämiseksi*

![Minion01 first](/h5/h5_a02.png)

- Minion01: tilat otettu käyttöön

![Webminion01 first](/h5/h5_a04.png)

- Webminion01: tilat otettu käyttöön

![Minion01 second](/h5/h5_a03.png)

- Minion01: tilat ovat jo käytössä

![Webminion01 second](/h5/h5_a05.png)

- Webminion01: tilat ovat jo käytössä

**Todennus:**

Varmistin vielä webminonin weppisivun toimivan ja minion01 ssh-yhteyden onnistuvan master koneelta.

![Manual check](/h5/h5_a06.png)

- ```curl 192.168.2.11``` - weppiminon tarjoilee oikean sivun
- ```ssh control@192.168.2.12``` - avain toimii kirjautumiseen
- ```sudo ufw status``` - control on sudokäyttäjä, hashattynä määritelty salasana toimii, ja UFW on oikeassa tilassa

## Lähteet

Tehtävä: 2025. Karvinen, T. [h5 Miniprojekti](https://terokarvinen.com/palvelinten-hallinta/#h5-miniprojekti)

Jaolim. [projektin repositorio](https://github.com/jaolim/salt-admin-setup)

Saltproject. [The Top File](https://docs.saltproject.io/en/3006/ref/states/top.html)

Saltproject. [salt.states.user](https://docs.saltproject.io/en/3006/ref/states/all/salt.states.user.html)