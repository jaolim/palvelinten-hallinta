# h2 Soitto kotiin

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

*Luo kahden orjan ja yhden mestarin ympäristön, jossa virtuaalikoneiden välinen kommunikaatio tapahtuu yksityisessä virtuaaliverkossa.*
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
  
  config.vm.define "slave002" do |slave002|
    slave002.vm.hostname = "slave002"
    slave002.vm.network "private_network", ip: "192.168.2.12"
    slave002.vm.provision "shell", path: "provision/slave.sh"
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

### Karvinen 2014: [Hello Salt Infra-as-Code](https://terokarvinen.com/2024/hello-salt-infra-as-code/)

Salttiin voi luoda omia tiloja, jotka ajetaan orjille komennolla ```sudo salt '*' state.apply tilamNimi```.

Moduulit tallennetaan polkuun ```/srv/salt/tilamNimi/``` ja niiden sisältö määritellään tiedostoon ```/srv/salt/tilamNimi/init.sls```.

Määrittelyssä käytetään [YAML syntaksia](https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml).

Esimerkki init.sls (hello tiedoston tulee olla olemassa polussa ```/tmp/hello```):

```
/tmp/hello:
  file.managed
```

### Salt contributors: [Salt overview](https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml)

#### Rules of YAML

- Oletus renderer monille saltin käyttämille tiedostoille.
- Data on esitetty ```avain: arvo``` pareissa.
- Välit *spacella* ei *tabillä*.
- Kommentit alkavat risuaidalla.

#### YAML simple structure
##### Scalar

```
# avain: arvo

vihannes: herne
hedelmä: omena
```

##### List

```
# ketjun_avain:
#  - arvo1
#  - arvo2

vihannekset:
  - herne
  - porkkana
hedelmät:
  - omena
  - appelsiini
```

##### Dictionary

```
lounas:
  juoma: vesi
  ruokalajit:
    - pihvi
	- perunat
	- salaatti
```

#### Lists and dictionaries - YAML block structures

Rakenne koostuu blokeista, joissa alemmat elementit sisennetään.

Sisennykset tehdään aina välilyönnillä ja sisennysten määrää ei ole määritelty, mutta 2 välilyöntiä on standardi.

## a) Hei infrakoodi!

*Kokeile paikallisesti (esim 'sudo salt-call --local') infraa koodina. Kirjota sls-tiedosto, joka tekee esimerkkitiedoston /tmp/ -kansioon.*

Loin kahden orjan ja yhden mestarin ympäristön ajamalla ```vagrant up``` ja ```vagrant reload``` komennot käyttäen *konfiguraatio* kohdassa listattuja asetuksia.

Varmistin orja-avain pyyntöjen tulleen *master* koneelle ja hyväksyin ne.

Määrittelin *hello* moduulin luomalla sille kansion polkuun ```/srv/salt/hello/``` ja luomalla sinne *init.sls* tiedoston, jossa määrittelin missä *hello* tiedoston tulee olla.

Tämän jälkeen ajoin tilan lokaalisti ````sudo salt-call --local state.apply hello``` komennolla ja varmistin tiedoston löytyvän

![Hello local](/h3/h3_a01.png)

**Ajankäyttö:** 5 minuuttia.

## b) Aja esimerkki sls-tiedostosi verkon yli orjalla.

Ajoin *hellon* ensin orjalla *slave002* komennolla ```sudo salt slave002 state.apply hello```, jonka jälkeen ajoin sen kaikilla orjilla komennolla ```sudo salt '*' state.apply hello```

![Hello slave](/h3/h3_b01.png)

*Toisella ajokerralla tiedosto löytyi jo slave002:lta, joten mitään ei tarvinnut tehdä.*

Varmistin vielä tiedoston löytyvän *slave001:ltä* käskemällä sitä ajamaan *ls* komennon.

![Hello exists](/h3/h3_b02.png)

**Ajankäyttö:** 4 minuuttia

## c) Laajennettu ls-tiedosto

*Tee sls-tiedosto, joka käyttää vähintään kahta eri tilafunktiota näistä: package, file, service, user. Tarkista eri ohjelmalla, että lopputulos on oikea. Osoita useammalla ajolla, että sls-tiedostosi on idempotentti.*

Olin ja aiemmassa tehtävässä määritellyt custom tilan, joka varmistaa, että apache on asennettuna ja käynnissä. Päätin muokata tästä version, joka muuttaa myös apachen oletussivun

Varmistin ensin, että pelkkä apachen asentava versio toimii.

![Apache works](/h3/h3_c01.png)

Asensin ensin Apachen *master* koneeseen ja tarkistin oletussivun sijainnin olevan polussa ```/var/www/html/index.html```.

Tämän jälkeen muutin *init.sls* muotoon:

```
apache2:
  pkg.installed: []
  service.running:
    - require:
	  - pkg: apache2
/var/www/html/index.html:
  file.managed
```
*Tämä versio tarkistaa ainoastaan, että index.html tiedosto on olemassa.* 

Ajoin muokatun version ja varmistin toimivuuden.

Muokkasin *init.sls:n* myös ylikirjoittamaan kotisivun sisällön:

```
apache2:
  pkg.installed: []
  service.running:
    - require:
      - pkg: apache2
/var/www/html/index.html:
  file.managed:
    - replace: True
    - contents:
      - "Heippahei!"
```

Ajoin tämän ensin *slave001:lle*, ja tämän toimittua ajoin sen kaikille orjille ```sudo salt '*' state.apply apache```.

![Apache final](/h3/h3_c02.png)

*Slave001 antoi nopean vastauksen, koska mitään ei tarvinnut tehdä ja slave002 vastasi asennettuaan paketit*

Komensin vielä orjat ajamaan ```sudo salt '*' state.single cmd.run 'curl localhost'``` todentaakseni kotisivun sisällön muuttuneen.

![Curl localhost](/h3/h3_c03.png)

**Ajankäyttö:** 45 minuuttia.

## Lähteet

#### Tehtävä

Karvinen 2025. [h3 Infraa koodina](https://terokarvinen.com/palvelinten-hallinta/#h3-infraa-koodina)

#### Tiivistettävät

Karvinen 2014: [Hello Salt Infra-as-Code](https://terokarvinen.com/2024/hello-salt-infra-as-code/)

Salt contributors: [Salt overview](https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml)

### Lisenssi

Sivun sisältöä saa levittää GPL-3.0 lisenssin sallimin ehdoin: https://github.com/jaolim/palvelinten-hallinta/tree/main?tab=GPL-3.0-1-ov-file