gitlab-windows
==============

Aantekeningen om GitLab in binnen een windows omgeving te draaien. Als je hier komt en je de ilusie hebt dat GitLab native kan draaien binnen windows...... dat kan dus niet. Hier wordt beschreven hoe een virtuele machine met GitLab opgezet kan worden.

Ik kies er bewust voor om niet de [Bitnami image](https://bitnami.com/stack/gitlab) te gebruiken. Een groot nadeel van deze image is dat het geen standaard Ubuntu installatie is. Na een korte test kwam ik er achter dat het root account standaard aan staat en dat tabcomplete in de shell niet werkt.

Als ik een stap niet benoem, dan is de default waarde voldoende.

Het volgen van de stappen zijn voldoende om gitlab op een veilige manier intern binnen een organisatie te draaien. Hiervoor is geen diepgaande kennis van Ubuntu nodig. Er staan hier en daar wel links waar je uitgebreide informatie kan vinden.

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
De CD is kan uit het station verwijderd worden door rechtsonder in het VM kader met de rechtermuisknop het CD icoontje te klikken. Kies 'Verwijder schijf van virtuele station', forceer afkoppeling zorgt niet voor problemen. `<Volgende>`

# Ubuntu instellen
* Login met het account dat is aangemaakt

## Updates en taal pakketten installeren

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

Open 
 
  
\* Negeer de incomplete taal waarschuwing, in de jaren dat ik Ubuntu in het Nederlands draai ben ik geen problemen tegen gekomen. Bovendien zijn de commando's en instellingen niet vertaald. Deze instelling zorgt ook voor de juiste localisatie instellingen.  
\*\* Oké, deze is wel lelijk vertaald :)  
\*\*\* Onthoudt deze

# Troubleshooting
* De Ubuntu VM heeft een zwart scherm
 * Druk op `CTRL + F1` om naar tty1 te gaan
