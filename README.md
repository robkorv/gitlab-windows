gitlab-windows
==============

Aantekeningen om GitLab in een windows omgeving te draaien.

Ik kies er bewust voor om niet de [Bitnami image](https://bitnami.com/stack/gitlab) te gebruiken. Een groot nadeel van deze image is dat het geen standaard Ubuntu installatie is. Na een korte test kwam ik er achter dat het root account standaard aan staat en dat tabcomplete in de shell niet werkt.

* Installeer [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* Download [Ubuntu 12.04 Netboot](http://nl.archive.ubuntu.com/ubuntu/dists/precise-updates/main/installer-amd64/current/images/netboot/mini.iso)
* Maak in VirtualBox een nieuwe virtuele machine aan
** Naam: Ubuntu GitLab
** Type: Linux
** Versie: Ubuntu (64 bit)
** Geheugengrootte: Minstens 1 gig
** Harde schijf: Maak nieuwe virtuele harde schijf nu aan
** Bestandstype harde schijf: VDI
** Opslag op fysieke harde schijf: Dynamisch gealloceerd
** Grootte virtuele schijf: 17 GB (dit is hetzelde als de Bitnami image)
* Rechtermuisknop op de zojuist aangemaakte virtuele machine -> Instellingen
