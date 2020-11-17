# Palvelinten hallinta: Harjoitustehtävät 3

**d) Näytä omalla git-varastollasi esimerkit komennoista ‘git log’, ‘git diff’ ja ‘git blame’. Selitä tulokset.**

Luodaan uusi GIT-versionhallinnan säilytyspaikka eli kansio  
$ git init varasto

Siirrytään kansioon ja konfiguroidaan varasto siten, että tehdään tallennus "git commit" vain kun varastoon on tehty muutoksia:  
$ cd varasto  
$ git add . && git commit ; git pull && git push

Tehdään Nanolla lueminut-tiedosto ja kirjoitetaan siihen tekstiä. Tallennetaan muutokset.
$ nano lueminut  
$ git add .  
$ git commit

Toistetaan kirjoita ja tallenna prosessia muutaman kerran ja viimeisellä kierroksella annettaan add . komento, mutta ei committia. Annetaan git log komento, joka näyttää eri commit komentojen ajankohdat ja kohdetiedoston  
$ git log

![logi](https://github.com/bgj377/palvelimetharjoitus3/blob/main/gitlog.JPG)

Seuraavaksi annetaan diff komento, joka näyttää mitä muutoksia lueminut-tiedostoon on tehty add . komennolla viimeisen commit komennon jälkeen.  
$ git diff

![diffi](https://github.com/bgj377/palvelimetharjoitus3/blob/main/gitdiff.JPG)

Viimeisenä annetaan blame komento, joka näyttää viimeisen muutoksen, joka tiedoston lueminut kullekkin riville tehtiin  
$ git blame lueminut

![blamemi](https://github.com/bgj377/palvelimetharjoitus3/blob/main/gitblame.JPG)

**e) Tee tyhmä muutos gittiin, älä tee commit:tia. Tuhoa huonot muutokset ‘git reset –hard’. Huomaa, että tässä toiminnossa ei ole peruutusnappia.**

Kirjoitetaan virheellistä tekstiä nanolla ja annetaan add . komento  
$ nano lueminut  
:Virheellistä tekstiä  
:Lisää virheellistä tekstiä  
$ git add .

Pyyhitään virheellinen teksti komennolla  
$ git reset --hard

![hardi](https://github.com/bgj377/palvelimetharjoitus3/blob/main/hard2.JPG)

**f) Tee uusi salt-moduli. Voit asentaa ja konfiguroida minkä vain uuden ohjelman: demonin, työpöytäohjelman tai komentokehotteesta toimivan ohjelman. Käytä tarvittaessa ‘find -printf “%T+ %p\n”|sort’ löytääksesi uudet asetustiedostot**

Asennan ensin käsin ja sitten modulin avulla Samba file serverin, joka on ohjelma, joka mahdollistaa tiedostojen jakamisen eri käyttöjärjestelmien välillä.

Asennetaan päivitykset ja asennetaan Samba  
$ sudo apt-get update  
$ sudo apt-get install samba

Tehdään kotihakemistoon kansio, jonka kautta tiedostoja voidaan jakaa  
$ mkdir ~/sambashare/

Lisätään Samban konfigurointitiedostoon jakohakemiston polku. Kirjoitetaan konfigurointitiedostoon seuraavat asiat
$ sudoedit /etc/samba/smb.conf  
:path = ~/sambashare/  
:read only = no  

Jotta konfiguraatiot saadaan otettua käyttöön, uudelleenkäynnistetään Samba  
$ sudo systemctl restart smbd

Tarkistetaan toimiiko Samban smbd demoni  
$ sudo service smbd status

Halutaan, että moduli hallitsee konfigurointitiedostoa, joten kopioidaan se saltin kansioon /srv/salt  
$ sudo cp /etc/samba/smb.conf /srv/salt/

Poistetaan Samban asennus  
$ sudo apt-get purge samba

Tehdään Samban asennus modulin avulla. Tehdään moduulille tiedosto.  
$ sudo mkdir -p /srv/salt/samba/ 

Luodaan init.sls moduli, joka asentaa Samban. Modulin avulla salt-kansiossa oleva smb.conf tiedoston määritetään ohittamaan Samban oma /etc/samba/smb.conf konfigurointitiedosto. Lisäksi moduli havainnoi, jos tiedostoon /etc/samba/smb.conf tehdään muutoksia, jolloin se lataa itsensä uudelleen, jotta salt-kansiossa olevan konfigurointitiedoston määritykset pysyvät voimassa. Lopuksi uudelleenkäynistetään Samba.

$ sudoedit /srv/salt/samba/init.sls  
samba:  
&nbsp; &nbsp; pkg.installed

smbd:  
&nbsp; &nbsp; service.running:  
&nbsp; &nbsp; &nbsp; &nbsp; -- require:  
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; -- pkg: samba  
&nbsp; &nbsp; &nbsp; &nbsp; -- watch:  
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; -- file: /etc/samba/smb.conf  

/etc/samba/smb.conf:  
&nbsp; &nbsp; file.managed:  
&nbsp; &nbsp; &nbsp; &nbsp; -- source: salt://smb.conf

sambaservice:  
&nbsp; &nbsp; service.running:  
&nbsp; &nbsp; &nbsp; &nbsp; -- name: smbd  
&nbsp; &nbsp; &nbsp; &nbsp; -- restart: True  
    
Otetaan init.sls käyttöön kaikille orjille  
$ sudo salt '*' state.apply samba


