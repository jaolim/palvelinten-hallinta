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

- **Käyttöjärjestelmä:** Debian 12 64-bit

## x) Lue ja tiivistä

### Karvinen 2021: [Two Machine Virtual Network With Debian 11 Bullseye and Vagrant](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/)

*(Huomaa: nykyinen Debian stable on 12-Bookworm, Vagrantissa "debian/bookworm64". Vanhentunutta 11-bullseye:ta ei enää käytetä)*

Vagrant on työkalu virtuaaliympäristöjen asentamisen automatisointiin.

Ympäristöjen asetukset konffataan *vagrantfile* tiedoston kautta ja komento ```$ vagrant up``` käynnistää siinä määritellyt virtuaalikoneet kun se ajetaan samassa kansiossa.

**Komentoja**
- ```$ vagrant ssh hostname``` - Ottaa yhteyden tiettyyn virtuaalikoneeseen (hostname = virtuaalikoneelle määritelty hostname).
- ```$ vagrant destroy``` - Tuhoaa kaikki virtuaalikoneet ja niiden tiedostot.

Asennus Vagrantille on alustakohtainen. Linuxille se löytyy suoraan apt-get paketinhallinnasta komennolla ```$ sudo apt-get install vagrant``` ja Windows tai Mac ympäristössä asennus tapahtuu [hashicorpin sivuilta ladattavan binäärin kautta](https://developer.hashicorp.com/vagrant/install).

**vagrantfile - tärkeimmät kohdat**

```
Vagrant.configure("2") do |config| # ("2") - API versio
  config.vm.box = "debian/bookworm64" # virtualisoitava käyttöjärjestelmä
	
  config.vm.define "t001" do |t001| # määriteltävä virtuaalikone
    t001.vm.hostname = "t001" # virtuaalikoneen hostname
  end
	
  config.vm.define "t002" do |t002| # toinen virtuaalikone
  t002.vm.hostname = "t002"
  end
end

```
Jo tämä on toimiva versio *vagrantfilestä*, mutta monet paikat, kuten IP-osoitteet jäävät määriteltäviksi oletusarvoilla. Alkuperäisestä artikkelista löytyy kattavampi versio.

### Karvinen 2018: [Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20quickstart%20salt%20stack%20master%20and%20slave%20on%20ubuntu%20linux)

*(Huomaa: Nykyisin ennen Saltin asentamista on asennettava ensin varasto [package repository], ohje h1 vinkeissä)*

Voidakseen asentaa saltin linuxiin täytyy ensin rekisteröidä sen paketin varasto luotetuksi lähteeksi. [Saltproject.io:n ohjeet tähän](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html).

Saltin komentorakenne koostuu mestarikoneesta, joka antaa ohjeita orjakoneille. Orjan täytyy tietää, missä mestari on ja mestarin täytyy hyväksyä orja-avain orjalta.

**Mestarin asennus**

```
master$ sudo apt-get update
master$ sudo apt-get -y install salt-master
master$ hostname -I
10.0.0.88
```

**Orjan asennus ja konffaus**

```
slave$ sudo apt-get update
slave$ sudo apt-get -y install salt-minion

slave$ sudoedit /etc/salt/minion

master: 10.0.0.88
id: tero
```

**Avaimen hyväksyntä**

```
master$ sudo salt-key -A
Unaccepted Keys:
tero
Proceed? [n/Y]
Key for minion tero accepted.
```

**Testaus**
```
master$ sudo salt '*' cmd.run 'whoami'
tero:
 root
```

Nyt masteri voi antaa orjalleen/orjilleen salt komentoja.

### Karvinen 2023: [Salt Vagrant - automatically provision one master and two slaves](https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file)

(*vain kohdat: Infra as Code - Your wishes as a text file, top.sls - What Slave Runs What States*)

#### Infra as Code

Salt tiloja voi määritellä tiedostoihin käyttäen [YAML-syntaksia](https://yaml.org/), jonka jälkeen tila voidaan käskea toteuttamaan.

```
$ sudo mkdir -p /srv/salt/hello
$ sudoedit /srv/salt/hello/init.sls
```

```
$ cat /srv/salt/hello/init.sls
/tmp/infra-as-code:
  file.managed

$ sudo salt '*' state.apply hello
```

#### top.sls

Top tiedosto määrittää mikä tavoitetila milläkin orjalla on.

```
$ sudo salt '*' state.apply hello^C
$ sudoedit /srv/salt/top.sls
$ cat /srv/salt/top.sls
base:
  '*':
    - hello
```

```$ sudo salt '*' state.apply```

## a) Hello Vagrant!

*Osoita jollain komennolla, että Vagrant on asennettu (esim tulostaa vagrantin versionumeron). Jos et ole vielä asentanut niitä, raportoi myös Vagrant ja VirtualBox asennukset. (Jos Vagrant ja VirtualBox on jo asennettu, niiden asennusta ei tarvitse tehdä eikä raportoida uudelleen.)*

Olin jo asentanut Vagrantin Windows ympäristöön ongelmitta.

![Vagrant version](/h2/h2_a01.png)

## b) Linux Vagrant

*Tee Vagrantilla uusi Linux-virtuaalikone.*

Aloitin tekemällä hakemiston virtuaalikoneilleni ja luomalla sinne *vagrantfilen* käyttäen komentoa ```vagrant init```.

![Vagrant initialization](/h2/h2_b01.png)

Tämän jälkeen poistin turhat kommentit ja lisäsin luotavan koneen tiedot *vagrantfileen*.

```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.define "master" do |master|
    master.vm.hostname = "master"
  end

end
```
*Vagrantfilen sisältö*

Ajoin ```vagrant up``` komennon, ja kun kone oli luotu otin siihen ssh yhteyden komennolla ```vagrant ssh master``` ja kokeilin nettiyhteyden pingaamalla googlen DNS palvelinta.

![Vagrant VM install](/h2/h2_b03.png)

*Asennus huomautti epäturvallisesta avaimesta ja vaihtoi avainparin, mutta tämä on odotettua käyttäytymistä ensimmäisellä kirjautumisella, koska vagrant käyttää julkisesti saatavilla olevaa avainta voidakseen kirjautua ensimmäistä kertaa, jonka jälkeen vaihtaa sen automaattisesti.*

*Asennus huomautti myös vanhasta guest editions versiosta.*

**Ajankäyttö:** 23 minuuttia.
 
## c) Kaksin kaunihimpi

*Tee kahden Linux-tietokoneen verkko Vagrantilla. Osoita, että koneet voivat pingata toisiaan.*

Tuhosin edellisen tehtävän virtuaalikoneen komennolla ```vagrant destroy -f``` ja muokkasin vagrantfileä luomaan 2 konetta, joissa IP-osoitteet oli määritelty.

```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.define "master" do |master|
    master.vm.hostname = "master"
	master.vm.network "private_network", ip: "192.168.2.1"
  end
  
  config.vm.define "slave001" do |slave001|
    slave001.vm.hostname = "slave001"
	slave001.vm.network "private_network", ip: "192.168.2.2"
  end

end
```

*Päivitetty vagrantfile.*

Ajoin ```vagrant up``` komennon ja asennuksen jälkeen otin molempiin koneisiin yhteydet ```vagrant ssh master``` ja ```slave001``` komennoilla.

Pingasin molemmilta koneilta toisiaan varmistaakseni yhteyden toimivuuden.

![Ping](/h2/h2_c01.png)

**Ajankäyttö:** 17 minuuttia.

## d) Herra-orja verkossa

*Demonstroi Salt herra-orja arkkitehtuurin toimintaa kahden Linux-koneen verkossa, jonka teit Vagrantilla. Asenna toiselle koneelle salt-master, toiselle salt-minion. Laita orjan /etc/salt/minion -tiedostoon masterin osoite. Hyväksy avain ja osoita, että herra voi komentaa orjakonetta.*


Rekisteröin salt repositorion luotettavaksi lähteeksi molempiin koneisiin käyttäen [saltproject.io:n ohjeita](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html).

```
$ sudo apt-get update
$ sudo apt-get install curl
$ mkdir -p /etc/apt/keyrings
$ curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
$ curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
```

Tämän jälkeen asensin *master* koneeseen *salt-masterin* ja *slave001* koneeseen *salt-minionin*.

- ```sudo apt-get install salt-master```
- ```sudo apt-get install salt-minion```

Määrittelin orjalle mestarin editoimalla minion tiedostoa ```sudoedit /etc/salt/minion``` lisäämällä sinne seuraavat rivit:

```
master: 192.168.2.1
id: slave001
```

Tämän jälkeen käynnistin orjapalvelun uudestaan ```$ sudo systemctl restart salt-minion.service```.

Kokeilin hyväksyä orja-avaimen mestarina, mutta huomasin, ettei avainpyyntö orjalta ollut mennyt läpi

![No key request](/h2/h2_d01.png)

Varmistin ensin *salt-minion* palvelun olevan käynnissä lokaalisti ajetulla salt komennoolla.

![Minion is running](/h2/h2_d02.png)

Tämän jälkeen kokeilin uudestaan IP-osoitteen pingaamista ja tämän onnistuttua lähdin hakemaan ratkaisua googlesta.

Ratkaisua tätä kautta ei suoraan löytynyt, mutta [saltprojektin dokumentaatiota selatessa](https://docs.saltproject.io/salt/install-guide/en/latest/topics/configure-master-minion.html#configure-master-minion) tuli vastaan maininta YAML:sta, jonka perusteella keksin tarkistaa config tiedoston sisennykset.

Päivitin tiedostoon lisätyt rivit seuraavaan muotoon:

```
  master: 192.168.2.1
  id: slave001
```

Tämän jälkeen ajoin *restart* komennon uudestaan ja hyväksyin onnistuneesti lähteneen avainpyynnön *master* koneen puolelta.

![Key accepted](/h2/h2_d03.png)

Varmistin vielä orjan vastaavan komentoihin.

![Answer](/h2/h2_d04.png)

**Ajankäyttö:** 52 minuuttia.

### Päivitys 09.04.2025

Tehtävässä koneiden välisen yhteyden korjannut askel ei ilmeisesti korjannut mitään, ja yhteys korjaantui muusta, sattumanvaraisesta syystä. Tehtävää tehdessä korjaus tuntuikin omituiselta, mutten silloin miettinyt asiaa enempää kunnes joku palautteista mainitsi samasta asiasta.

Päädyin lähtemään selvittämään ongelman syitä, mutta tein tämän pienissä pätkissä muiden useiden päivien aikana, joten en kirjoittanut reaaliaikaisesti raporttia, vaan tämä on ongelman ratkaisun jälkeen kirjoitettu tiivistelmä selvityksen vaiheista.

Itse tehtävää tehdessä pidin ensimmäisen yhteysongelman ilmetessä tauon, jolloin olin pois koneelta noin tunnin ajan virtuaalikoneiden ollessa käynnissä, ja oletan, että tämä jostain syystä korjasi yhteyden, eikä muokattu orjakonfiguraatio.
Varmaa syytä yhteyden korjautumiselle tehtävässä en tiedä.

**Vianselvitys**

- Tuhosin ja uudelleenloin virtuaalikoneet ja varmistin yhteysongelman tapahtuvan uudestaan.
- Varmistin *tcpdumpilla* pingien vastaanottamisen```sudo apt-get install tcpdump``` & ```sudo tcpdump -i eth1 icmp```.
- Kuuntelin *tcpdumpilla* oikeaa verkkokorttia ja varmistin, että orja yrittää ottaa yhteyttä uudelleenkäynnistettäessä ```sudo tcpdump -i eth1```.
-- Johtopäätös: orja yrittää ottaa yhteyttä ja tämä saapuu perille, mutta yhteydenotto hylätään jostain syystä.
- Varmistin MAC osoitteen päivittyvän oikein ARP tableen: ```cat /proc/net/arp```.
-- Toinen kone tulee näkyviin pingin tai muun yhteydenoton jälkeen.

Tässä kohtaan päätin kokeilla aktivoida VirtualBoxin Promiscuous moden virtuaalikoneille. Tämän aktivoinnin ja uudelleenkäynnistyksen jälkeen yhteys toimi.

Promiscuous mode antaa virtuaalikoneen vastaanottaa muiden koneidem MAC osoitteisiin lähetettyjä paketteja, joten en tiedä miksei yhteys toiminut ilman sitä.

Automatisoin vielä promiscuous moden aktivoinnin ja eri pakettien asennukset vagrantilla ja testasin toimintaa.
Koneet nousivat VirtualBoxin mukaan promiscuous mode päällä, mutta yhteysongelma oli edelleen ja sen korjaaminen vaati koneiden uudelleen käynnistyksen.

Nyt koneet siis toimivat kun ajetaan:

```
vagrant up
vegrant reload
```

**Päivitetyt tiedostot**

Vagrantfile:

```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  

  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.2.10"
	config.vm.provider "virtualbox" do |vb|
		vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
	end
	master.vm.provision "shell", path: "provision/master.sh"
  end
  
  config.vm.define "slave001" do |slave001|
    slave001.vm.hostname = "slave001"
    slave001.vm.network "private_network", ip: "192.168.2.11"
	config.vm.provider "virtualbox" do |vb|
		vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
	end
	slave001.vm.provision "shell", path: "provision/slave.sh"
  end
  
end
```

master.sh:

```
sudo apt-get install curl -y
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get install salt-master -y
```

slave.sh:

```
sudo apt-get install curl -y
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get install salt-minion -y
echo "master: 192.168.2.10" | sudo tee -a /etc/salt/minion
sudo systemctl restart salt-minion
```


## e) Kokeile vähintään kahta tilaa verkon yli

*(viisikosta: pkg, file, service, user, cmd)*

Käskin tiedoston *exist* olla olemassa *file.managed* komentoa käyttäen ja varmistin tämän olevan totta orjakoneen terminaalista.

Tämän jälkeen ajoin saman komennon uudestaan varmistaen idempotenssin, jonka jälkeen kerroin, ettei tiedostoa tulisi olla *file.absent* käskyllä.

![file commands](/h2/h2_e01.png)

Seuraavaksi päätin määritellä tiedostoon tilan, joka installoi ja käynnistää apachen, jos sitä ei ole asennettuna tai se ei ole käynnissä.

Tähän löytyi suoraan ohje [ansible-cn.readthedocs.io sivuilta](https://ansible-cn.readthedocs.io/en/stable/topics/tutorials/starting_states.html), josta piti ainoastaan muuttaa apache muotoon apache2.

![Define and apply state](/h2/h2_e02.png)

init.sls sisältö:

```
apache2:
  pkg.installed: []
  service.running:
    - require:
      - pkg: apache2
```

![Changes](/h2/h2_e03.png)

*Huomionarvoista on, ettei käynnistystä tarvinnut tehdä, koska apache2 käynnistyy oletusarvoisesti asennuksen yhteydessä.*

Ajoin vielä *apply.staten* kertaalleen todentaakseni, ettei muutoksia tarvinnut tehdä.

![No changes](/h2/h2_e04.png)

Varmistin vielä apachen oletussivun toimivan käskemällä orjaa ajamaan *curl* komennon *localhostiin*.

![curl results](/h2/h2_e05.png)

**Ajankäyttö:** 51 minuuttia

## Lähteet

#### Tehtävä

Karvinen, T. 2025. [h2 Soitto kotiin](https://terokarvinen.com/palvelinten-hallinta/#h2-soitto-kotiin).

#### Lyhennettävät

Karvinen 2021: [Two Machine Virtual Network With Debian 11 Bullseye and Vagrant](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/)

Karvinen 2018: [Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20quickstart%20salt%20stack%20master%20and%20slave%20on%20ubuntu%20linux)

Karvinen 2023: [Salt Vagrant - automatically provision one master and two slaves](https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file)

#### Tiedonhaku

Ansible-cn.readthedocs.io. [Tutorial - Starting States](https://ansible-cn.readthedocs.io/en/stable/topics/tutorials/starting_states.html)

Github.com/hashicorp. [Vagrant insecure key detected](https://github.com/hashicorp/packer/issues/3293).

Saltproject.io. [Salt install Linux Debian](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html)

Yaml.org. https://yaml.org/

### Lisenssi

Sivun sisältöä saa levittää GPL-3.0 lisenssin sallimin ehdoin: https://github.com/jaolim/palvelinten-hallinta/tree/main?tab=GPL-3.0-1-ov-file