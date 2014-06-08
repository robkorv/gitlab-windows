gitlab-windows
==============

Aantekeningen om GitLab in binnen een windows omgeving te draaien. Als je hier komt en je de ilusie hebt dat GitLab native kan draaien binnen windows...... dat kan dus niet. Hier wordt beschreven hoe een virtuele machine met GitLab opgezet kan worden.

Ik kies er bewust voor om niet de [Bitnami image](https://bitnami.com/stack/gitlab) te gebruiken. Een groot nadeel van deze image is dat het geen standaard Ubuntu installatie is. Na een korte test kwam ik er achter dat het root account standaard aan staat en dat tabcomplete in de shell niet werkt.... wie weet wat er nog meer anders is?

Ook kies ik niet voor de nieuwste Ubuntu LTS release omdat deze niet wordt ondersteund door GitLab op het moment van schrijven. Ubuntu 12.04 LTS, wordt ondersteund tot 26 April 2017 en is dus een veilige keuze.

Als ik een stap niet benoem, dan is de default waarde voldoende.

Het volgen van de stappen is voldoende om gitlab op een veilige manier intern binnen een organisatie te draaien. Hiervoor is geen diepgaande kennis van Ubuntu nodig. Er staan hier en daar wel links waar je uitgebreide informatie kan vinden.

TIP: De terminal van Ubuntu heeft tab-completion. Meestal is 2 á 3 letters van een commando, directory of bestand gevolgd door een `TAB` voldoende.

TODO:
- [ ] Ip range beveiligen
- [x] https activeren
- [ ] backup binnen windows (via smb?) http://doc.gitlab.com/ce/raketasks/backup_restore.html
- [ ] server onderhoud beschrijven (update herinnering via mail? updaten via http?)
- [ ] http://doc.gitlab.com/ce/raketasks/features.html

__LET OP!__: Dit bestand is niet compleet

# VM Install
* Installeer [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* Download [Ubuntu 12.04 Netboot](http://nl.archive.ubuntu.com/ubuntu/dists/precise-updates/main/installer-amd64/current/images/netboot/mini.iso)
* Maak in VirtualBox een nieuwe virtuele machine aan
  * Naam: Ubuntu GitLab
  * Type: Linux
  * Versie: Ubuntu (64 bit)
  * Geheugengrootte: Minstens 1 gig
  * Harde schijf: Maak nieuwe virtuele harde schijf nu aan
  * Bestandstype harde schijf: VDI
  * Opslag op fysieke harde schijf: Dynamisch gealloceerd
  * Grootte virtuele schijf: 17 GB (dit is hetzelde als de Bitnami image)
* Rechtermuisknop op de zojuist aangemaakte virtuele machine -> Instellingen
  * Netwerk -> Netwerk bridge adapter (anders is de vm niet extern bereikbaar)

# Ubuntu Install  
* Start de aangemaakte virtuele machine
* Blader naar de gedownloade mini.iso om deze als opstartschijf te gebruiken
* Druk op `ENTER` terwijl de cursor op 'Install' staat
* Installatie
  * Language: Dutch\*
  * Toetsenbordindeling detecteren?: Nee
  * Oorsprong van het toetsenbord: Engels (US)
  * Toetsenbordindeling: Engels (US) - Engels (US, alternatief internationaal)
  * Computernaam: gitlab
  * Land van Ubuntu-archief-spiegelserver\*\*: Nederland
  * Wat is de volledige naam van de nieuwe gebruiker?: GitLab
  * Wat is de gebruikersnaam voor uw account?: gitlab\*\*\*
  * Wat is het wachtwoord voor de nieuwe gebruiker?: \<sterk wachtwoord\>\*\*\*

Bij de volgende opties zijn de default waarden genoeg. Op een gegeven moment wordt het basis systeem geinstalleerd.
Kies bij:

* Welke software wilt u installeren?
 * Basic Ubuntu Server
 * OpenSSH Server

Hierna zijn de defaults ook voldoende. 
De CD kan uit het station verwijderd worden door rechtsonder in het VM kader met de rechtermuisknop het CD icoontje te klikken. Kies 'Verwijder schijf van virtuele station', forceer afkoppeling zorgt niet voor problemen. `<Volgende>`

Log na de reboot vervolgens in met het aangemaakt account.

# firewall instellen

[Uitgebreide informatie firewall instellen](https://help.ubuntu.com/12.04/serverguide/firewall.html)

Ik ga hem zo instellen dat alle poorten dicht zitten behalve voor ssh en https.

```
# firewall aan zetten
sudo ufw enable

# ssh accepteren
sudo ufw allow ssh

# https accepteren
sudo ufw allow https
```

# Ubuntu updaten

```bash
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get install $(check-language-support)
```

## Netwerk instellen

Voor de volgende stappen is enige basis kennis van VIM nodig, ik vermeld één keer de commando's en short-cuts om een bestand te wijzigen. Wil je VIM onder de knie krijgen lees dan [A Byte of Vim](http://www.swaroopch.com/notes/vim/)

[Uitgebreide informatie netwerk instellen](https://help.ubuntu.com/12.04/serverguide/network-configuration.html)

Ga bij je netwerkbeheerder na welk ip je mag gebruiken. Het ip van je gateway en dns server(s) heb je ook nodig. Gebruik de volgende commando's om deze informatie uit Ubuntu te halen.

```bash
# eth0 is het externe ip adres
ifconfig

# gateway
route -n

# dns server(s)
cat /etc/resolv.conf
```

Open `/etc/network/interfaces`

```bash
sudo vim /etc/network/interfaces
```

Druk op `i` om INSERT mode te activeren. Maak je wijzigingen.  
Druk op `ESC` om weer naar COMMAND mode te gaan.  
Druk op `:` om een commando te geven. `wq` (write quit), slaat het bestand op en sluit VIM.  
`q!` sluit VIM zonder de wijzigingen op te slaan.

Verander de informatie onder `# The primary network interface` met de informatie over je eigen netwerk.

* ip: 192.168.1.10
* subnet mask: 255.255.255.0
* gateway: 192.168.1.254
* dns: 192.168.1.254, 195.241.77.55, 195.241.77.58

```ini
auto eth0
iface eth0 inet static
    address 192.168.1.10
    netmask 255.255.255.0
    gateway 192.168.1.254
    dns-nameservers 192.168.1.254 195.241.77.55 195.241.77.58
```

Na de wijziging breng je eth0 down en weer up

```bash
sudo ifdown eth0
sudo ifup eth0
```

Test je netwerk met

```bash
ping -c 10 www.google.nl
```

Test ook of de VM te bereiken is met de windows bak `WINDOWSTOETS + R` -> `cmd`

```cmd
ping <ip van de vm>
```

Vanaf nu kan je ook via ssh verbinden. Dit werk makkelijker omdat je niet meer hoeft te switchen naar de terminal op de VM. [Putty](http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe) is een simpele stabiele ssh client.

Open putty, voer het ip van de VM in en druk op `Open`. Accepteer de key en login met het account waarmee je de VM hebt geinstalleerd.

Je kan in de huide VM uitloggen door `exit` te typen.

# GitLab installeren

GitLab heeft een mailserver nodig om mail te versturen.

```bash
sudo tasksel install mail-server
```

Kies bij de installatie voor Internetsite.

Download [GitLab Ubuntu 12.04 LTS 64bit](https://www.gitlab.com/downloads/). Vervang de onderstaande link met de laatste versie

```bash
wget https://downloads-packages.s3.amazonaws.com/ubuntu-12.04/gitlab_6.8.1-omnibus.4-1_amd64.deb
```

Installeer gdebi zodat er een dependency check wordt uitgevoerd voor de installatie.

```bash
sudo apt-get install gdebi-core
```

GitLab installeren

```bash
sudo gdebi gitlab_6.8.1-omnibus.4-1_amd64.deb
sudo gitlab-ctl reconfigure
```

GitLab is nu geinstalleerd en bereikbaar via http. Open het ip van je VM in de browser, de standaard admin login  is username `root` and password `5iveL!fe`.

GitLab configureren

```bash
sudo touch /etc/gitlab/gitlab.rb
sudo chmod 600 /etc/gitlab/gitlab.rb
```

Open `sudo vi /etc/gitlab/gitlab.rb`, om het externe url op te geven.

```ruby
external_url "http://192.168.1.10"
```

Voer `sudo gitlab-ctl reconfigure` uit om de wijzegingen door te voeren.

Https aanzetten

Zelf gesigneerde certificaten zijn standaard aanwezig in Ubuntu. Je kan natuurlijk je eigen genereren/installeren. Hoe dat moet lees je bij de uitgebreide informatie

[Uitgebreiden certificaat informatie](https://help.ubuntu.com/12.04/serverguide/certificates-and-security.html)

Open `sudo vi /etc/gitlab/gitlab.rb` en wijzig de regels voor https 

```ruby
external_url "https://192.168.1.10"
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/ssl/certs/ssl-cert-snakeoil.pem"
nginx['ssl_certificate_key'] = " /etc/ssl/private/ssl-cert-snakeoil.key"
```

Voer `sudo gitlab-ctl reconfigure` uit om de wijzegingen door te voeren.
  
\* Negeer de incomplete taal waarschuwing, in de jaren dat ik Ubuntu in het Nederlands draai ben ik geen problemen tegen gekomen. Bovendien zijn de commando's en instellingen niet vertaald. Deze instelling zorgt ook voor de juiste localisatie instellingen.  
\*\* Oké, deze is wel lelijk vertaald :)  
\*\*\* Onthoudt deze

# Troubleshooting
* De Ubuntu VM heeft een zwart scherm
 * Druk op `CTRL + F1` om naar tty1 te gaan
* Openssl geeft een `unable to write 'random state'` melding
  * `sudo rm .rnd`
