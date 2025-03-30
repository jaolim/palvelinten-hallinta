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

![Machine information](/h1/h1_a01.png)

*Koneen tiedot, joista ID:t on piilotettu.*

## b) Asenna Salt (salt-minion) Linuxille (uuteen virtuaalikoneeseesi)

Koska salttia ei löydy suoraan apt-get paketinhallinnasta, pitää tämä paketti lisätä sinne manuaalisesti.

Noudatin tässä [saltproject.io:n ohjeita](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html) ja asennuksen jälkeen varmistin toimivuuden.

![Keyrings directory & key download](/h1/h1_b01.png)

- ```$ mkdir...``` - Keyrings hakemiston luonti
- ```$ curl...``` - Avaimen lataus

![Apt repo target configuration & install](/h1/h1_b02.png)

- ```$ curl... ``` - Kohderepositorion määritys aptille
- ```$ sudo apt-get...``` - Asennus

![Version check](/h1/h1_b03.png)

- Version tarkistus asennuksen onnistumisen varmistamiseksi

**Ajankäyttö:** 19 minuuttia, josta noin puolet meni raportin kirjoittamiseen.

## c) Viisi tärkeintä

*Näytä Linuxissa esimerkit viidestä tärkeimmästä Saltin tilafunktiosta: pkg, file, service, user, cmd. Analysoi ja selitä tulokset.*

### pkg

Päätin käyttää salttia TLDR:n asentamiseen. TLDR(Too long, didn't read) tarjoaa pikakäyttöoppaat moniin Linux ohjelmiin.

![tldr doesn't exist](/h1/h1_c01.png)

- ```$ tldr salt```-komennolla varmistin, ettei tldr ole asennettuna

![tldr should exist](/h1/h1_c02.png)

- ```$ sudo salt-call``` - Ajetaan salt käsky
- ```--local``` - Ajetaan lokaalista
- ```-l info``` - Log level ja määritys tasoksi info
- ```state.single``` - Tarkastellaan yhtä tilaa
- ```pkg.installed``` - Ohjelman tulee olla asennettuna
- ```tldr``` - Ohjelman nimi

![tldr salt](/h1/h1_c03.png)

*Edellisen kuvan login loppu ja saltin tldr sivu*

![tldr does exist](/h1/h1_c04.png)

Sama komento uudelleenajettuna meni läpi onnistuneesti, muttei asentanut tldr:ää uudestaan, koska se oli jo asennettuna.

### file

Ajoin komennon ```sudo salt-call --local -l info state.single file.managed ~/Desktop/h1/toBe.txt contents="be"``` varmistettuani ettei kohdehakemistosta löydy *toBe.txt* tiedostoa.
Tämän jälkeen varmistin että tiedosto löytyy oikealla sisällöllä.

![File should be](/h1/h1_c05.png)
- ```file.managed``` - Tiedoston tulee olla olemassa
- ```contents="be"``` - Tiedoston tulee sisältää teksti be
- ```$ cat toBe.txt``` - Tulosta toBe.txt tiedoston sisältö

Ajoin vielä luontokomennon uudestaan, jonka jälkeen poistin tiedoston.

![File is](/h1/h1_c06.png)

- Tiedosto löytyy ja on oikeassa tilassa
- ```file.absent``` - Tiedostoa ei tulisi olla, ja tiedosto poistetaan

### service

Asensin alkuun Apache2:n komennolla ```$ sudo apt-get install apache2``` ja totesin sen olevan oletusarvoisesti päällä menemällä selaimella osoitteeseen *localhost:80*.

![running](/h1/h1_c07.png)

- ```$ sudo salt-call --local state.single service.running apache2 enable=True``` - Varmistin vielä käynnistykomennon toimivuuden

![dead](/h1/h1/c08.png)
*Tapoin palvelun ja varmistin tämän selaimella*

### user

Varmistin ettei käyttäjää *testaaja* ole olemassa.

![doesn't exist](/h1/h1_c09.png)

- ```user.absent testaaja``` - testaaja nimisen käyttäjän ei tule olla olemassa

Tämän jälkeen lisäsin käyttäjän *testaaja*.

![exists](/h1/h1_c10.png)

- ```user.present``` - testaaja nimisen käyttäjän tulee olla olemassa

![shouldn't exist](/h1/h1_c11.png)

*Poistin vielä käyttäjän ajamalla ensimmäisen komennon uudestaan*

### cmd

Cmd.run komento ajaa komentoja. Tämä ei ota lähtökohteisesti kantaa tilaan ennen komennon ajamista, joten käyttäjän täytyy pitää itse houli, että niin tehdään.

Päätin luoda tiedoston *~/Desktop/h1/test.txt* komennolla ```sudo salt-call --local state.single cmd.run 'touch ~/Desktop/h1/test.txt'```

![wrong directory](/h1/h1_c12.png)

*Ensimmäinen yritys päätyi virheeseen, koska komentoa ajetaan roottina, joten ~ ei ohjaa oikeaan kotihakemistoon*

![fixed path](/h1/h1/c13.png)

*Kirjoitettuani koko polun komento toimi, koska hakemisto löytyi*

**Ajankäyttö:** 1 tunti ja 20 minuuttia.

## d) Idempotentti

*Anna esimerkki idempotenssista. Aja 'salt-call --local' komentoja, analysoi tulokset, selitä miten idempotenssi ilmenee.*

Idempotenttinen komento tekee muutoksia vain, jos järjestelmä ei ole jo valmiiksi halutussa tilassa. Aikaisemmassa tehtävässä osoitin tätä ajamalla osan komennoista jo toivotussa tilassa, ja tulosteista ilmeni, ettei mitään tarvinnut tehdä.

Cmd kohdan esimerkkini taas on sellainen, joka toistaa komennon aina ajaettaessa, ja jätinkin sen korjaamisen idempotenttiseksi tätä osuutta varten.

![not working](/h1/h1_d01.png)

*Ajoin vielä edellisen komennon todentaakseni, ettei tämä ole idempotentti.*

Tämän jälkeen korjasin idempotenssin lisäämällä *creates* säännön korjaaman tämän.

![working](/h1/h1_d01.png)

- ```creates="/home/janne/Desktop/h1/test.txt"``` - Katsoo onko tiedosto olemassa ja ajaa komennon ainoastaan, jos näin ei ole.

Koska cmd:ssä joutuu idempotenssin toteuttamaan itse, on parempi käyttää komentoja, joissa se on hoidetta valmiiksi, ja turvautua cmd:iin ainoastaan, kun muilla keinoin ei komentoa saa ratkaistua.

![working but why](/h1/h1_d02.png)

```sudo salt-call --local state.single cmd.run 'echo Hello World!' creates="/home/janne/Desktop/h1/test2.txt"```

Muokattu versio komennosta tulostaa "Hello World!" *stdouttiin*, jos määriteltyä tiedostoa ei ole olemassa.

Tämä on periaatteessa idempotentti, mutta sen toiminnallisuus melko järjetöntä, ja se toimii lähinnä esimerkkinä siitä, että idempotenssilla on arvoa ainoastaan jos se on määritelty mielekkäästi.

**Ajankäyttö:** 25 minuuttia.
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