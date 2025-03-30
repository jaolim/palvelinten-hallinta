# h1 Viisikko

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

- **CPU:** 4 Core

- **RAM:** 8 GB

- **Kiintolevy:** 200GB

## x) Lue ja tiivistä

*Tässä x-alakohdassa ei tarvitse tehdä testejä tietokoneella, vain lukeminen tai kuunteleminen ja tiivistelmä riittää. Tiivistämiseen riittää muutama ranskalainen viiva. Ei siis vaadita pitkää eikä essee-muotoista tiivistelmää. Lisää kuhunkin jokin oma kysymys tai huomio.*

### Karvinen 2006: [Raportin Kirjoittaminen](https://terokarvinen.com/2006/06/04/raportin-kirjoittaminen-4/)

Sisällöltään raportin tulee kattaa ympäristö, jossa tehtävä suoritettiin, sekä tehtävän askeleet tarpeeksi tarkasti, että tehtävä on toistettavissa raportin pohjalta.

Ulkoasultaan raportin tulee olla selkeä hyödyntäen väliotsikoita ja huolellista kieltä.

Lähteet on listattava raporttiin.

Pahoja virheitä raportissa ovat sepittäminen, plagiointi ja tekijänoikeuksien rikkominen esimerkiksi kuvien luvattomalla käytöllä.

### Karvinen 2018: [Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux](https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/)

Salt on ohjelma, jolla yksi mestaritietokone(master) voi kontrolloida useita orjatietokoneita(slave).

**Master asennus**

```
master$ sudo apt-get -y install salt-master
```

- Mestari tarvitsee 4505/tcp ja 4506/tcp portit auki.

**Orjan asennus**

```
slave$ sudo apt-get -y install salt-minion
```

- Orjan täytyy tietää missä sen mestari on.

**Orja-avaimen hyväksyminen**

```
master$ sudo salt-key -A
```

**Komentojen antaminen**

```
master$ sudo salt '*' cmd.run 'whoami'
```

### Karvinen 2023: [Run Salt Command Locally](https://terokarvinen.com/2021/salt-run-command-locally/)

Salt komentoja voi ajaa lokaalisti käyttäen ```salt-call --local``` komentoa, joka on hyödyllistä esimerkiksi testaukseen ja harjoitteluun.

Samat komennot toimivat eri käyttöjärjestelmissä, ja kuten komentojen nimeämisestä huomaa, ne tyypillisesti kuvailevat tilaa, jossa koneen tulee olla.
Tämä tarkoittaa, komennot ajetaan vain, jos kone ei ole valmiiksi oikeassa tilassa, eli nämä komennot ovat idempotentteja.
Komennoissa, jotka eivät ole idempotentteja, tulee idempontenssista huolehtia itse.

**Salt orjan asennus ja version tarkistus**

```
$ sudo apt-get update
$ sudo apt-get -y install salt-minion
$ sudo salt-call --version
```

**pkg.installed** - *Ohjelman tulee olla installoitu*

```
$ sudo salt-call --local -l info state.single pkg.installed tree

$ sudo salt-call --local -l info state.single pkg.removed tree
```

- Idempotentti

- Asentaa ohjelman, jos tämä ei ole asennettuna

**file.managed** - *Tiedoston tulee olla olemassa*

```
$ sudo salt-call --local -l info state.single file.managed /tmp/hellotero

$ sudo salt-call --local -l info state.single file.managed /tmp/moitero contents="foo"

$ sudo salt-call --local -l info state.single file.absent /tmp/hellotero
```

- Idempotentti

- Luo tiedoston, jos tiedostoa ei ole olemassa

**service.running** - *Demonin tulee olla käynnissä*

```
$ sudo salt-call --local -l info state.single service.running apache2 enable=True

$ sudo salt-call --local -l info state.single service.dead apache2 enable=False
```

- Idempotentti

- Käynnistää demonin, jos demoni ei ole käynnissä

**user.present** - *Käyttäjän tulee olla olemassa*

```
$ sudo salt-call --local -l info state.single user.present terote08

$ sudo salt-call --local -l info state.single user.absent terote08
```

- Idempotentti

- Luo käyttäjän, jos käyttäjää ei ole olemassa

**cmd.run** - *Aja komento*

```
$ sudo salt-call --local -l info state.single cmd.run 'touch /tmp/foo' creates="/tmp/foo"
```

- Ei idempotentti

- Ajaa komennon

- Idempotenssista huolehdittava itse. 

**Ohjeet**

```
$ sudo salt-call --local sys.state_doc
```

## Asenna Debian 12-Bookworm virtuaalikoneeseen

*Poikkeuksellisesti tätä alakohtaa ei tarvitse raportoida, jos siinä ei ole mitään ongelmia. Mutta jos on ongelmia, sitten täsmällinen raportti, jotta voidaan ratkoa niitä yhdessä.*

Asensin Debianin virtuaalikoneeseen ongelmitta raportin alussa listatuilla spekseillä.

## b) Asenna Salt (salt-minion) Linuxille (uuteen virtuaalikoneeseesi)

Koska salttia ei löydy suoraan apt-get paketinhallinnasta, pitää tämä paketti lisätä sinne manuaalisesti.

Noudatin tässä [saltproject.io:n ohjeita](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html) ja asennuksen jälkeen varmistin toimivuuden.

![Keyrings directory & key download](/h1/h1_a01.png)

- ```$ mkdir...``` - Keyrings hakemiston luonti
- ```$ curl...``` - Avaimen lataus

![Apt repo target configuration & install](/h1/h1_a02.png)

- ```$ curl... ``` - Kohderepositorion määritys aptille
- ```$ sudo apt-get...``` - Asennus

![Version check](/h1/h1_a03.png)

- Version tarkistus asennuksen onnistumisen varmistamiseksi

**Ajankäyttö:** 19 minuuttia, josta noin puolet meni raportin kirjoittamiseen.

## c) Viisi tärkeintä

*Näytä Linuxissa esimerkit viidestä tärkeimmästä Saltin tilafunktiosta: pkg, file, service, user, cmd. Analysoi ja selitä tulokset.*



## Idempotentti

*Anna esimerkki idempotenssista. Aja 'salt-call --local' komentoja, analysoi tulokset, selitä miten idempotenssi ilmenee.*


## Lähteet

### Tehtävänanto

Karvinen, T. 2025. [h1 Viisikko.](https://terokarvinen.com/palvelinten-hallinta/#h1-viisikko)

### Tiivistettävät

Karvinen, T. 2006. [Raportin Kirjoittaminen](https://terokarvinen.com/2006/06/04/raportin-kirjoittaminen-4/)

Karvinen, T. 2018. [Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux](https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/)

Karvinen, T. 2023. [Run Salt Command Locally](https://terokarvinen.com/2021/salt-run-command-locally/)

### Tiedonhaku

Saltproject.io. [salt-call dokumentaatio.](https://docs.saltproject.io/en/3006/ref/cli/salt-call.html)

Saltproject.io. [Salt asennusohje](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html)

### Lisenssi

Sivun sisältöä saa levittää GPL-3.0 lisenssin sallimin ehdoin: https://github.com/jaolim/palvelinten-hallinta/tree/main?tab=GPL-3.0-1-ov-file