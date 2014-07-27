gitlab-windows
==============

Aantekeningen om GitLab in binnen een windows omgeving te draaien. Als je hier komt en je de ilusie hebt dat GitLab native kan draaien binnen windows...... dat kan dus niet. Hier wordt beschreven hoe een virtuele machine met GitLab opgezet kan worden.

Ik kies er bewust voor om niet de [Bitnami image](https://bitnami.com/stack/gitlab) te gebruiken. Een groot nadeel van deze image is dat het geen standaard Ubuntu installatie is. Na een korte test kwam ik er achter dat het root account standaard aan staat en dat tabcomplete in de shell niet werkt.... wie weet wat er nog meer anders is?

Als ik een stap niet benoem, dan is de default waarde voldoende.

Het volgen van de stappen is voldoende om gitlab op een veilige manier intern binnen een organisatie te draaien. Hiervoor is geen diepgaande kennis van Ubuntu nodig. Er staan hier en daar wel links naar waar je uitgebreide informatie kan vinden.

TIP: De terminal van Ubuntu heeft tab-completion. Meestal is 2 á 3 letters van een commando, directory of bestand gevolgd door een `TAB` voldoende.

TODO:
- [ ] Ip range beveiligen
- [x] https activeren
- [ ] backup binnen windows (via smb?) http://doc.gitlab.com/ce/raketasks/backup_restore.html
- [x] server onderhoud beschrijven (update herinnering via mail? updaten via http?) https://help.ubuntu.com/12.04/serverguide/automatic-updates.html, apticron werkt, unattended-upgrades nog niet
- [x] root als mail voldoende is, vanwege de forward instellingen

__LET OP!__: Dit bestand is niet compleet

# VM Install
* Installeer [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* Download [Ubuntu 14.04 Netboot](http://nl.archive.ubuntu.com/ubuntu/dists/trusty-updates/main/installer-amd64/current/images/netboot/mini.iso)
* Maak in VirtualBox een nieuwe virtuele machine aan ([GitLab Hardware Requirements])
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
  * Language: Dutch
  * Toetsenbordindeling detecteren?: Nee
  * Oorsprong van het toetsenbord: Engels (US)
  * Toetsenbordindeling: Engels (US) - Engels (US, alternatief internationaal)
  * Computernaam: gitlab
  * Land van Ubuntu-archief-spiegelserver: Nederland
  * Wat is de volledige naam van de nieuwe gebruiker?: GitLab
  * Wat is de gebruikersnaam voor uw account?: gitlab
  * Wat is het wachtwoord voor de nieuwe gebruiker?: \<sterk wachtwoord\>
  * Schijfindelingsmethode: Begeleid - benut gehele schijf
  * Wilt u deze aanpassingen wegschrijven naar de schijven: Ja

Bij de volgende opties zijn de default waarden genoeg. Op een gegeven moment wordt het basis systeem geinstalleerd.
Kies bij:

* Welke software wilt u installeren?
 * Basic Ubuntu Server
 * OpenSSH Server

Hierna zijn de defaults ook voldoende. 
De CD kan uit het station verwijderd worden door rechtsonder in het VM kader met de rechtermuisknop het CD icoontje te klikken. Kies 'Verwijder schijf van virtuele station', forceer afkoppeling zorgt niet voor problemen. `<Volgende>`

Log na de reboot vervolgens in met het aangemaakt account.

## Firewall instellen

[Uitgebreide informatie firewall instellen](https://help.ubuntu.com/12.04/serverguide/firewall.html)

Ik ga hem zo instellen dat alle poorten dicht zitten behalve voor ssh en https.

```bash

# ssh accepteren
sudo ufw allow ssh

# https accepteren
sudo ufw allow https

# http accepteren
sudo ufw allow http

# firewall aan zetten
sudo ufw enable
```

## Ubuntu updaten

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

```
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

# GitLab install

## Mailserver instellen

GitLab heeft een mailserver nodig om mail te versturen.

```bash
sudo tasksel install mail-server
```

Kies bij de installatie voor Internetsite.

Het root account krijgt nog wel eens interne mail, deze kan geforward worden door `root: <email@adres.nl>` toe te voegen aan `/etc/aliases`.

```bash
# mail adres instellen
sudo vi /etc/aliases

# nieuwe alias aanmelden
sudo newaliases
```

Ook de aangemaakte GitLab gebruiker krijgt wel eens interne mail. Deze kan geforward worden door in `~/.forward` een mail adres te zetten

```bash
vi ~/.forward
```

## GitLab downloaden en installeren

##GitLab installeren

Download [GitLab Ubuntu 14.04 LTS 64bit](https://www.gitlab.com/downloads/). Vervang de onderstaande link met de laatste versie

```bash
wget https://downloads-packages.s3.amazonaws.com/ubuntu-14.04/gitlab_7.1.0-omnibus-1_amd64.deb
```

Installeer gdebi zodat er een dependency check wordt uitgevoerd voor de installatie.

```bash
sudo apt-get install gdebi-core
```

```bash
sudo gdebi gitlab_7.1.0-omnibus-1_amd64.deb
sudo gitlab-ctl reconfigure
```

De standaard admin login is username `root` and password `5iveL!fe`.

# GitLab configureren

```bash
sudo touch /etc/gitlab/gitlab.rb
sudo chmod 600 /etc/gitlab/gitlab.rb
```

## Https aanzetten

Zelf gesigneerde certificaten zijn standaard aanwezig in Ubuntu. Je kan natuurlijk je eigen genereren/installeren. Hoe dat moet lees je bij de uitgebreide informatie

[Uitgebreide certificaat informatie](https://help.ubuntu.com/12.04/serverguide/certificates-and-security.html)

Open `sudo vi /etc/gitlab/gitlab.rb` en voeg de volgende info toe

```ruby
external_url "https://192.168.1.10"
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/ssl/certs/ssl-cert-snakeoil.pem"
nginx['ssl_certificate_key'] = " /etc/ssl/private/ssl-cert-snakeoil.key"
```

Voer `sudo gitlab-ctl reconfigure` uit om de wijzigingen door te voeren.

# Ubuntu onderhouden

## Automatische updates 

[Uitgebreide automatische updates informatie](https://help.ubuntu.com/12.04/serverguide/automatic-updates.html)

Unattended-upgrades installeren, dit zorgt ervoor dat security updates automatisch worden geinstalleerd.

```bash
sudo apt-get install unattended-upgrades
```

De instellingen staan in `/etc/apt/apt.conf.d/10periodic`.  
Met de volgend instelling wordt de package list elke dag geupdate en security updates geinstalleerd. Om de 7 dagen worden oude niet geinstalleerd packages verwijderd.

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

Je kan op de hoogte gehouden worden door unattended-upgrades via de mail.

```bash
sudo vi /etc/apt/apt.conf.d/50unattended-upgrades
```

Haal de `//` weg bij `Unattended-Upgrade::Mail "root@localhost";` en vul het mail adres in waar de status mails naar toe moeten.

## Update notificaties

Unattended-upgrades mailt alleen als er security updates zijn geinstalleerd. Apticron doet dit voor alle updates.

```bash
sudo apt-get install apticron
```

Stel het mail adres in

```bash
sudo vi /etc/apticron/apticron.conf
```

Verander `EMAIL="root@example.com"` naar het mail adres dat je wilt gebruiken.

# GitLab onderhoud

## GitLab status

```bash
sudo gitlab-ctl status
```

## GitLab check

```bash
sudo gitlab-rake gitlab:check
```

De melding `Install the init script` kan je negeren, aangezien GitLab al automatisch start.

## Start/Stop/Restart GitLab 

```bash
# Start all GitLab components
sudo gitlab-ctl start

# Stop all GitLab components
sudo gitlab-ctl stop

# Restart all GitLab components
sudo gitlab-ctl restart

# Restart individual components
sudo gitlab-ctl restart sidekiq
```

# Backup

Ik zie geen reden om Ubuntu zelf te backuppen. Dit is niet nodig omdat alle belangrijke bestanden in een GitLab backup zit. Mocht de VM verloren gaan dan is er met minimale configuratie een nieuwe op te zetten waar een een GitLab backup in hetsteld kan worden.

## GitLab backup maken

De backup wordt aangemaakt in `/var/opt/gitlab/backups/`.

```bash
sudo gitlab-rake gitlab:backup:create
```

## GitLab dagelijkse backups inplannen

Het volgende zorgt ervoor dat er elke dag om 02:00 en backup wordt gemaakt.

```bash
sudo crontab -e
```

> Als de editor wordt gevraagd, kies dan `/usr/bin/vim.basic` voor VIM.

Voeg de regel `0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create` toe.

## GitLab backups automatisch opschonen

Met de instelling `gitlab_rails['backup_keep_time'] = 604800` in `/etc/gitlab/gitlab.rb` worden backups ouder dan 7 dagen verwijderd bij de volgende backup actie. De tijd is in seconden.

# Troubleshooting
* De Ubuntu VM heeft een zwart scherm
 * Druk op `CTRL + F1` om naar tty1 te gaan
* Openssl geeft een `unable to write 'random state'` melding
  * `sudo rm .rnd`
* Mailtjes komen niet aan
 * Grote kans dat ze als spam worden gezien

[GitLab Hardware Requirements]: https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/requirements.md#hardware-requirements
