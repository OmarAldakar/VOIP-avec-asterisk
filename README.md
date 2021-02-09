# Asterisk
Dans ce projet on configure un PBX (commutateur téléphonique privé) à l'aide d'asterisk.

## Installation d'Asterisk sur Ubuntu 18.04/16.04

Traduction du tuto trouver ici : https://computingforgeeks.com/how-to-install-asterisk-16-lts-on-ubuntu-18-04-16-04-debian-9/

### Etape 1 : Mettre à jour son système
On commence par mettre à jour son système pour ne pas avoir de soucis au niveau des dépendance 

```
sudo apt update && sudo apt -y upgrade
sudo reboot
```
### Etape 2 : Installer les dépendances d'Asterisk 16 LTS
Une fois le système mis à jour, installez tous les packages de dépendance que nécessite Asterisk :
```
sudo apt -y install git curl wget libnewt-dev libssl-dev libncurses5-dev subversion  libsqlite3-dev build-essential libjansson-dev libxml2-dev  uuid-dev
```
Si vous obtenez une erreur pour le package subversion sur Ubuntu comme ci-dessous:
```
E: Package 'subversion' has no installation candidate
```
Ajoutez ensuite le dépôt universe et installez subversion à partir de celui-ci:
```
sudo add-apt-repository universe
sudo apt update && sudo apt -y install subversion
```
### Etape 3 : Télécharger Asterisk 16 LTS
On va effectuer une installation depuis les sources, on les télécharges : 

```
cd /usr/src/
sudo curl -O http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-16-current.tar.gz
```
Ensuite on les extrait : 
```
sudo tar xvf asterisk-16-current.tar.gz
cd asterisk-16*/
```
Exécutez la commande suivante pour télécharger la bibliothèque du décodeur mp3 dans l'arborescence source.
```
sudo contrib/scripts/get_mp3_source.sh
```
Assurez-vous que toutes les dépendances sont résolues:
```
sudo contrib/scripts/install_prereq install
```
Vous devriez obtenir un message de réussite à la fin:
```
#############################################
## install completed successfully
#############################################
```
### Etape 4 : Générer et installer Asterisk 16 
Exécutez le script de configuration pour satisfaire les dépendances de génération.
```
sudo ./configure
```
Un message de succès devrait s'afficher comme ceci :
```
..................
configure: Menuselect build configuration successfully completed

               .$$$$$$$$=..      
            .$7$7..          .7$7:.    
          .$:.                 ,$7.7   
        .$7.     7$$           .$77  
     ..$.       $$$            .$$7 
    ..7$   .?.   $$$   .?.       7$$.
   $.$.   .$$7. $$7 .7$$.      .$$.
 .777.   .$$$77$$77$$$7.      $$,
 $$~      .7$$$$$$$7.       .$$.
.$7          .7$$$$7:          ?$$.
$$          ?7$$$$$I        .$$7 
$$       .7$$$$$$$$      :$$. 
$$       $$$7$$$$$$    .$$.  
$$        $$   7$$7  .$$    .$$.   
$$             $$7         .$$.    
7$$7            7$$        7$$      
 $$$                        $$       
  $$7.                       $  (TM)     
   $$$$.           .7$$$  $      
     $$$$$$7$$$$$.$$$      
       $$$$$$$$.                

configure: Package configured for: 
configure: OS type  : linux-gnu
configure: Host CPU : x86_64
configure: build-cpu:vendor:os: x86_64 : pc : linux-gnu :
configure: host-cpu:vendor:os: x86_64 : pc : linux-gnu :
```
On peut choisir des options de configuration en lançant la commande suivante : 
```
sudo make menuselect
```
On génère ensuite l'éxécutable d'Asterisk en lançant la commande suivante : 

```
sudo make
```

Une fois terminé, installez Asterisk en exécutant la commande:
```
sudo make install
```

### Créer un utilisateur Asterisk 
Dans cette partie il s'agit de créer un utilisateur pour les services Asterisk afin de lui donner les bonnes permissions pour assurer l'intégrité de notre système.
```


sudo groupadd asterisk
sudo useradd -r -d /var/lib/asterisk -g asterisk asterisk
sudo usermod -aG audio,dialout asterisk
sudo chown -R asterisk.asterisk /etc/asterisk
sudo chown -R asterisk.asterisk /var/{lib,log,spool}/asterisk
sudo chown -R asterisk.asterisk /usr/lib/asterisk
```
Ensuite on assign l'utilisateur asterisk qu'on vient de créer comme utilisateur par défaut du service Asterisk
```
$ sudo vim /etc/default/asterisk
AST_USER="asterisk"
AST_GROUP="asterisk"

$ sudo vim /etc/asterisk/asterisk.conf
runuser = asterisk ; The user to run as.
rungroup = asterisk ; The group to run as.
```
On redémarre le service Asterisk pour que les changements soit pris en considération. 
```
sudo systemctl restart asterisk
```
On peut aussi activer le service asterisk au démarrage de notre système.
```
sudo systemctl enable asterisk
```
On peut enfin tester, 

```
$ sudo asterisk -rvv
Asterisk 16.0.1, Copyright (C) 1999 - 2018, Digium, Inc. and others.
Created by Mark Spencer <markster@digium.com>
Asterisk comes with ABSOLUTELY NO WARRANTY; type 'core show warranty' for details.
This is free software, with components licensed under the GNU General Public
License version 2 and other licenses; you are welcome to redistribute it under
certain conditions. Type 'core show license' for details.
=========================================================================
Running as user 'asterisk'
Running under group 'asterisk'
Connected to Asterisk 16.0.1 currently running on ubuntu-01 (pid = 10154)
ubuntu-01*CLI> core  show channels
Channel              Location             State   Application(Data)             
0 active channels
0 active calls
0 calls processed
ubuntu-01*CLI> exit
Asterisk cleanly ending (0).
Executing last minute cleanups
```
## Configuration de son PBX 
Il faut configurer son NAT et ouvrir le port 5080 ainsi que les port 10 000 à 20 000 (pour pouvoir laisser entrer les pacquet UDP). Pour cela il va falloir que notre addresse IP soit statique. 

Avec la commande `ifconfig `, nous pourront récupérer toute les informations que l'on nécessite pour pouvoir compléter les fichiers de configuration suivant : 

Le fichier rtp.conf : 
```
;
; RTP Configuration
;
[general] 
;
; RTP start and RTP end configure start and end addresses
;
rtpstart=10000
rtpend=20000
```

Le fichier sip.conf : 
```
[general]
language=fr
context=others
nat=force_rport,comedia
qualify=yes
directmedia=no
udpbindaddr=____________
host=dynamic
bindport =5080
localnet=____________
externip=______________
canreinvite=no

[1001]
type=friend
context=from-internal
host=dynamic
secret=__________
disallow=all
allow=ulaw
nat=force_rport,comedia
qualify=yes

[1002]
type=friend
context=from-internal
host=dynamic
secret=_________
disallow=all
allow=ulaw
nat=force_rport,comedia
qualify=yes
```

Le fichier extension.conf : 
```
[from-internal]
exten=>_1XXX,1,Dial(SIP/${EXTEN},5)
exten=>_1XXX,2,VoiceMail(${EXTEN})
exten =>_1XXX,3,PlayBack(vm-goodbye)
exten =>_1XXX,4,HangUp()
exten=>_1XX,1,Dial(IAX2/${EXTEN},5)
exten=>_1XX,2,VoiceMail(${EXTEN})
exten =>_1XX,3,PlayBack(vm-goodbye)
exten =>_1XX,4,HangUp()
```


exten => 600,1,VoiceMailMain(${CALLERID(num)}@default)

### Configuration du répondeur
La configuration du répondeur (voicemail), se fait dans le fichier voicemail.conf
```
[general]
format=wav49|gsm|wav
maxsilence=10
silencethreshold=128
maxlogins=3
sendvoicemail=yes


[default]
1001=>,1001, *****************@****.com
1002=>,1002, *****************@****.com
serveremail= *****************@****.com
attach=yes
```

## Documentation et lien intéressant

Documentation officielle : https://wiki.asterisk.org/wiki/display/AST/Getting+Started
