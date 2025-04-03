# h2 Soitto kotiin

## x) Lue ja tiivistä

### Karvinen 2021: [Two Machine Virtual Network With Debian 11 Bullseye and Vagrant](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/)

*(Huomaa: nykyinen Debian stable on 12-Bookworm, Vagrantissa "debian/bookworm64". Vanhentunutta 11-bullseye:ta ei enää käytetä)*

Vagrant on työkalu virtuaaliympäristöjen asentamisen automatisointiin.

Ympäristöjen asetukset konffataan *vagrantfile* tiedoston kautta ja komento ```$ vagrant up``` käynnistää siinä määritellyt virutaalikoneet kun se ajetaan samassa kansiossa.

**Komentoja**
- ```$ vagrant ssh hostname``` - Ottaa yhteyden tietyyn virtuaalikoneeseen (hostname = virtuaalikoneelle määritelty hostname).
- ```$ vagrant destroy``` - Tuhoaa kaikki virtuaalikoneet ja niiden tiedostot.

Asennus Vagrantille on alustakohtainen. Linuxille se löytyy suoraan apt-get paketinhallinnsta komennolla ```$ sudo apt-get install vagrant``` ja Windows tai Mac ympäristössä asennus tapahtuu [hashicorpin sivuilta ladattavan binäärin kautta](https://developer.hashicorp.com/vagrant/install).

**vagrantfile - tärkeimmät kohdat**

```
Vagrant.configure("2") do |config| # ("2") - koneiden määrä
	config.vm.box = "debian/bookworm64" # virtualisoitava käyttöjärjestelmä
	
	config.vm.define "t001" do |t001| #määriteltävä virtuaalikone
		t001.vm.hostname = "t001" #virtuaalikoneen hostname
	end
	
	config.vm.define "t002" do |t002|
	t002.vm.hostname = "t002"
	end
end

```
Jo tämä on toimiva versio *vagrantfilestä*, mutta monet paikat, kuten IP-osoitteet määriteltäviksi oletusarvoilla. Alkuperäisestä artikkelista löytyy kattavampi versio.

### Karvinen 2018: [Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20quickstart%20salt%20stack%20master%20and%20slave%20on%20ubuntu%20linux)

*(Huomaa: Nykyisin ennen Saltin asentamista on asennettava ensin varasto [package repository], ohje h1 vinkeissä)*

### Karvinen 2023: [Salt Vagrant - automatically provision one master and two slaves](https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file)

(*vain kohdat: Infra as Code - Your wishes as a text file, top.sls - What Slave Runs What States*)

## a) Hello Vagrant!

*Osoita jollain komennolla, että Vagrant on asennettu (esim tulostaa vagrantin versionumeron). Jos et ole vielä asentanut niitä, raportoi myös Vagrant ja VirtualBox asennukset. (Jos Vagrant ja VirtualBox on jo asennettu, niiden asennusta ei tarvitse tehdä eikä raportoida uudelleen.)*

## b) Linux Vagrant

*Tee Vagrantilla uusi Linux-virtuaalikone.*

## c) Kaksin kaunihimpi

*Tee kahden Linux-tietokoneen verkko Vagrantilla. Osoita, että koneet voivat pingata toisiaan.*

## Herra-orja verkossa

*Demonstroi Salt herra-orja arkkitehtuurin toimintaa kahden Linux-koneen verkossa, jonka teit Vagrantilla. Asenna toiselle koneelle salt-master, toiselle salt-minion. Laita orjan /etc/salt/minion -tiedostoon masterin osoite. Hyväksy avain ja osoita, että herra voi komentaa orjakonetta.*

## e) Kokeile vähintään kahta tilaa verkon yli

*(viisikosta: pkg, file, service, user, cmd)*

