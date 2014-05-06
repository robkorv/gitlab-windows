gitlab-windows
==============

Aantekeningen om GitLab in een windows omgeving te draaien.

Ik kies er bewust voor om niet de [Bitnami image](https://bitnami.com/stack/gitlab) te gebruiken. Een groot nadeel van deze image is dat het geen standaard Ubuntu installatie is. Na een korte test kwam ik er achter dat het root account standaard aan staat en dat tabcomplete in de shell niet werkt.

Als ik een stap niet benoem, dan is de default waarde voldoende.

# Pre-install
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

# Post-install  
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
  
\* Negeer de incomplete taal waarschuwing, in de jaren dat ik Ubuntu in het Nederlands draai ben ik geen problemen tegen gekomen. Bovendien zijn de commando's en instellingen niet vertaald. Deze instelling zorgt ook voor de juiste localisatie instellingen.
\*\* Oké, deze is wel lelijk vertaald :)
