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

- **Käyttöjärjestelmä:** Debian 12 64-bit

## x) Lue ja tiivistä

### Karvinen 2021: [Two Machine Virtual Network With Debian 11 Bullseye and Vagrant](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/)

*(Huomaa: nykyinen Debian stable on 12-Bookworm, Vagrantissa "debian/bookworm64". Vanhentunutta 11-bullseye:ta ei enää käytetä)*

Vagrant on työkalu virtuaaliympäristöjen asentamisen automatisointiin.

Ympäristöjen asetukset konffataan *vagrantfile* tiedoston kautta ja komento ```$ vagrant up``` käynnistää siinä määritellyt virtuaalikoneet kun se ajetaan samassa kansiossa.

**Komentoja**
- ```$ vagrant ssh hostname``` - Ottaa yhteyden tietyyn virtuaalikoneeseen (hostname = virtuaalikoneelle määritelty hostname).
- ```$ vagrant destroy``` - Tuhoaa kaikki virtuaalikoneet ja niiden tiedostot.

Asennus Vagrantille on alustakohtainen. Linuxille se löytyy suoraan apt-get paketinhallinnsta komennolla ```$ sudo apt-get install vagrant``` ja Windows tai Mac ympäristössä asennus tapahtuu [hashicorpin sivuilta ladattavan binäärin kautta](https://developer.hashicorp.com/vagrant/install).

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

Salt komentoja voi määritellä tiedostoihin käyttäen [YAML-syntaksia](https://yaml.org/).

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

*Asennus huomautti epäturvallisesta avaimesta ja vaihtoi avainparin, mutta tämä on odettua käyttäytymistä ensimmäisellä kirjautumisella, koska vagrant käyttää julkisesti saatavilla olevaa avainta voidakseen kirjautua ensimmäistä kertaa, jonka jälkeen vaihtaa sen automaattisesti.*

*Asennus huomautti myös vanhasta guest editions versiosta.*

**Ajankäyttö:** 23 minuuttia.
 
## c) Kaksin kaunihimpi

*Tee kahden Linux-tietokoneen verkko Vagrantilla. Osoita, että koneet voivat pingata toisiaan.*

Tuhosin edellisen tehtävän virtuaalikoneen komennolla ```vagrant destroy -f``` ja muokkasin vagrantfileä luomaan 2 konetta, joissa ip-osoitteet oli määritelty.

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

Ajoin ````vagrant up``` komennon ja asennuksen jälkeen otin molempiin koneisiin yhteydet ```vagrant ssh master``` ja ````slave001``` komennoilla.

Pingasin molemmilta koneilta toisiaan varmistaakseni yhteyden toimivuuden.

[Ping](/h2/h2_c01.png)

**Ajankäyttö:** 17 minuuttia.

## Herra-orja verkossa

Rekisteröin salt repositorion luotettavaksi lähteeksi molempiin koneisiin käyttäen [saltproject.io:n ohjeita](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html).

```
$ sudo apt-get update
$ sudo apt-get install curl
$ mkdir -p /etc/apt/keyrings
$ curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
$ curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
```

Tämän jälkeen asensin *master* koneeseen *salt-masterin* ja *slave001* koneeseen *salt minionin*.

- ```sudo apt-get install salt-master```
- ```sudo apt-get install salt-minion```

Määrittelin orjalle mestarin editoimalla minion tiedostoa ```sudoedit /etc/salt/minion``` lisäämällä sinne seuraavat rivit:
````
master: 192.168.2.1
id: slave001

```

Tämän jälkeen käynnistin orjapalvelun uudestaan ```$ sudo systemctl restart salt-minion.service```.

Kokeilin hyväksyä orja-avaimen mestarina, mutta huomasin, ettei avainpyyntö orjalta ollut mennyt läpi

![No key request](/h2/h2_c02.png)

Varmistin ensin *salt-minion* palvelun olevan käynnissä lokaalisti ajetulla salt komennoolla.

![Minion is running](/h2/h2_c03.png)

Tämän jälkeen kokeilin uudestaan IP-osoitteen pingaamista ja tämän onnistuttua lähdin hakemaan ongelmaa googlesta.

Ratkaisua tätä kautta ei suoraan löytynyt, mutta [saltprojektin dokumentaatiota selatessa] tuli vastaan maininta YAML:sta, jonka perusteella keksin tarkistaa config tiedoston sisennykset.

Päivitin tiedoston seuraavaan muutoon:
````
  master: 192.168.2.1
  id: slave001

```

Tämän jälkeen ajoin *restart* komennon uudestaan ja hyväksyin onnistuneesti lähteneen avainpyynnön *master* koneen puolelta.

![Key accepted](/h2/h2_c04.png)

Varmistin vielä orjan vastaavan komentohin.

![Answer](/h2/h2_c05.png)

**Ajankäyttö:** 52 minuuttia.

*Demonstroi Salt herra-orja arkkitehtuurin toimintaa kahden Linux-koneen verkossa, jonka teit Vagrantilla. Asenna toiselle koneelle salt-master, toiselle salt-minion. Laita orjan /etc/salt/minion -tiedostoon masterin osoite. Hyväksy avain ja osoita, että herra voi komentaa orjakonetta.*

## e) Kokeile vähintään kahta tilaa verkon yli

*(viisikosta: pkg, file, service, user, cmd)*

## Lähteet

#### Tehtävä

Karvinen, T. 2025. [h2 Soitto kotiin](https://terokarvinen.com/palvelinten-hallinta/#h2-soitto-kotiin).

#### Lyhennettävät

Karvinen 2021: [Two Machine Virtual Network With Debian 11 Bullseye and Vagrant](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/)

Karvinen 2018: [Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20quickstart%20salt%20stack%20master%20and%20slave%20on%20ubuntu%20linux)

Karvinen 2023: [Salt Vagrant - automatically provision one master and two slaves](https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file)

#### Tiedonhaku

Github.com/hashicorp. [Vagrant insecure key detected](https://github.com/hashicorp/packer/issues/3293).

Saltproject.io. [Salt install Linux Debian](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html)

Yaml.org. https://yaml.org/

### Lisenssi

Sivun sisältöä saa levittää GPL-3.0 lisenssin sallimin ehdoin: https://github.com/jaolim/palvelinten-hallinta/tree/main?tab=GPL-3.0-1-ov-file