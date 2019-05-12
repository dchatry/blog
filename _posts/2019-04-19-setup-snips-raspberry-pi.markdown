---
layout: post
title: Mettre en place un assistant vocal hébergé sur un Raspberry Pi (FR)
date: 20:02 19-04-2019
headline: 
taxonomy:
    category: blog
    tag: [raspberry]
---

![header_snips_rpi_ha](/blog/assets/articles/images/header_snips_rpi_ha.png)



Cela faisait quelques temps que je voulais un assistant personnel à la maison mais l'idée que mes données fassent le tour du monde juste pour allumer une ampoule me donnait des boutons. Mais il y a peu je suis tombé sur [Snips](https://snips.ai/) un assistant vocal open-source installable sur tout un tas d'appareils, son intérêt principal étant que toutes les commandes exécutées le sont directement sur la machine et non sur un serveur lointain, d'ailleurs, Snips peut fonctionner et contrôler les appareils de votre réseau local sans être connecté à Internet ! Cerise sur le gâteau, Snips est une start-up française (cocorico !).

Le [Raspberry Pi](<https://www.raspberrypi.org/>) (RPi) qui traînait dans un de mes tiroirs fera un parfait cobaye pour tester tout ça. Pour le son, comme je n'ai pas d'enceinte ou haut-parleur à disposition, je vais utiliser une enceinte bluetooth.

Mon but final étant de pouvoir réaliser quelques commandes simples ("quel temps va-t-il faire ?", "quel heure il est ?"), de pouvoir contrôler mon aspirateur Xiaomi Roborock V2 ("passe l'aspirateur dans la cuisine", "retourne à ta base", etc.), et quelques appareils comme les lampes, la télé ou le thermostat. Nous verrons comment on peut mettre en place tout ça via [Home Assistant]([https://www.home-assistant.io](https://www.home-assistant.io/)), un outil open-source également permettant de contrôler tous les objets connectés de votre maison.



## Matériel requis

- [Raspberry Pi 3 B+](<https://www.kubii.fr/raspberry-pi/2119-raspberry-pi-3-modele-b-1-gb-kubii-713179640259.html>)
- [Respeaker 4 mic array](<https://www.reichelt.com/fr/de/respeaker-4-mic-array-fuer-raspberry-pi-rpi-resp-4mic-p248717.html?&trstct=pos_3>) (ou microphone standard, je recommande tout de même de sélectionner un des microphones sur cette liste pour de meilleures performances : <https://docs.snips.ai/articles/raspberrypi/hardware/microphones>)
- Une carte micro SD (32 Go ou plus) + adaptateur de branchement USB ou lecteur de carte
- Des hauts-parleurs AUX ou une enceinte bluetooth
- Un SSD (optionnel, mais recommandé car le système va procéder à beaucoup de lectures/écritures qui usent la carte SD à la longue)

![materiel](/blog/assets/articles/images/materiel.jpg)



## Installation de Hassbian

Nous allons commencer par télécharger **Hassbian**, un Rasbian modifié qui intègre directement la suite **Home Assistant**, pour ce faire :

1. Téléchargez l'image en suivant [ce lien] (https://github.com/home-assistant/pi-gen/releases/latest) (cliquez sur image_XXXX-XX-XX_Hassbian.zip)
2. Téléchargez [BalenaEtcher](<https://www.balena.io/etcher/>)
3. Lancez BalenaEtcher et sélectionnez le zip de l'image que vous venez de télécharger, puis la carte SD
4. Flashez !



## (Optionnel) Installation sur SSD

Pour installer Hassbian sur un SSD, il faut à la fois flasher l'image sur la carte micro SD et sur le SSD, appliquez donc les étapes ci-dessus sur les deux supports.

Une fois que votre carte micro SD est flashée, ouvrez la sur votre ordinateur (elle devrait se nommer "boot") et vous devriez trouver à la racine un fichier nommé `config.txt`.

Editez le et rajoutez cette ligne à la fin du fichier :

```
program_usb_boot_mode=1
```

Cette ligne va servir à activer le démarrage système sur les ports USB du RPi.

A présent, insérez la carte micro SD dans le RPi, et branchez le. Laissez le démarrer (une minute devrait suffire). Débranchez le RPi, enlevez la carte micro SD, puis branchez le SSD au RPi. Il devrait désormais démarrer sur le SSD directement !



## Activation du mode "headless"

Nous pourrions brancher le RPi à un moniteur et un clavier/souris pour effectuer toutes les configurations nécessaires, mais il est beaucoup plus pratique d'activer un accès à distance en ligne de commande.

Comment faire ? Rien de plus simple :

1. Branchez la carte micro SD ou le SSD à votre ordinateur
2. A la racine de "boot", créez un fichier nommé `ssh` _sans extension_
3. Dans le même dossier, créez un fichier nommé `wpa_supplicant.conf`
4. Editez le avec un éditeur de texte type "bloc-note" ou mieux [Notepad++](<https://notepad-plus-plus.org/fr/>)
5. Collez-y la configuration suivante en modifiant **MonWifi** et **MonMotDePasseWifi** :

```
country=fr
update_config=1
ctrl_interface=/var/run/wpa_supplicant

network={
 scan_ssid=1
 ssid="MonWifi"
 psk="MonMotDePasseWifi"
}
```

Remettez la carte micro SD ou le SSD dans votre RPi et branchez le. Il devrait se connecter tout seul au Wifi. Vous pouvez également simplement brancher un câble Ethernet et le relier à votre routeur.

Maintenant que le RPi est en mode "headless", il faut trouver son adresse IP. La façon la plus simple est simplement de se connecter à l'interface d'administration de votre routeur (en général 192.168.0.1 ou 192.168.1.1, cherchez la marque sur Internet si vous ne trouvez vraiment pas). Ensuite, rendez-vous dans la rubrique détaillant les périphériques connectés au réseau local, chez moi cette rubrique se nomme "Mon réseau local". Ici vous devriez pouvoir voir un périphérique nommé "**hassbian**" et son adresse IP correspondante :

![hassbian](/blog/assets/articles/images/hassbian.PNG)

Une fois l'adresse connue, connectez vous grâce à un client SSH, par exemple avec [PuTTY](<https://www.putty.org/>). Saisissez l'adresse IP dans "Host Name (or IP address)", dans "Port" saisissez 22 et cliquez sur "Open". Par défaut, le login de connexion est **pi** et le mot de passe **raspberry**.

Pour plus de sécurité, changez le mot de passe grâce à la commande :

```
passwd
```



## Installation de SNIPS

La façon la plus simple d'installer Snips sur le RPi est d'installer l'assistant **Sam** qui est un vrai couteau suisse en ligne de commande permettant d'effectuer [tout un tas d'opérations](<https://docs.snips.ai/reference/sam>) de routine sur Snips.

Pour l'installer vous aurez besoin d'installer **npm**, le gestionnaire de paquet de NodeJS, pour ce faire, ouvrez votre terminal, sous Windows ouvrez la console (touches Win + R, tapez **cmd** puis Entrée) puis collez la ligne suivante et faites Entrée :

```
sudo npm install -g snips-sam
```

Maintenant que **Sam** est installé vous pouvez vous connecter à votre RPi grâce à la commande suivante (remplacer par l'adresse IP de votre RPi, pour la retrouvez voir plus haut) :

```
sam connect 192.168.0.XX
```

puis lancez :

```
sam init
```

pour installer Snips sur le RPi.

Si `sam init` ne fonctionne pas, installez Snips manuellement via les commandes ci-dessous :

```
sudo apt-get update
sudo apt-get install -y dirmngr
sudo bash -c 'echo "deb https://raspbian.snips.ai/$(lsb_release -cs) stable main" > /etc/apt/sources.list.d/snips.list'
sudo apt-key adv --keyserver pgp.mit.edu --recv-keys D4F50CDCA10A2849
sudo apt-get update
sudo apt-get install -y snips-platform-voice
sudo apt-get install -y snips-template snips-skill-server
```



## Configuration audio 

Pour pouvoir parler et entendre notre assistant, il nous faut désormais configurer les haut-parleurs et le microphone de notre système. 

Commencez par vous connecter au RPi via SSH.

### Configuration bluetooth

Je recommande d'utiliser l'architecture **Alsa** qui est installée de base dans le noyau Linux et qui permet de contrôler les périphériques audio. Une autre solution consiste à installer **[Pulseaudio](<https://doc.ubuntu-fr.org/pulseaudio>)** qui devrait fonctionner également, si vous avez des soucis avec Alsa, essayez de l'installer.

Le paquet permettant de configurer une connexion audio bluetooth est **bluealsa**, pour l'installer :

```
sudo apt-get install bluealsa
```

Une fois installé, vous devriez pouvoir vous connecter à votre enceinte, commencez par éteindre le bluetooth des appareils qui vous entoure pour éviter qu'ils se connectent à l'enceinte.

Allumez l'enceinte en mode appairage et lancez les commandes suivantes : 

```
sudo bluetoothctl
scan on
```

Une liste d'appareils apparaît normalement avec chacun un adresse MAC associée. Repérez l'enceinte, par exemple pour mon enceinte Xiaomi, j'ai : 

```
[NEW] Device 00:00:01:04:2B:E0 MI Portable Bluetooth Speaker
```

Copiez l'adresse MAC, pour moi c'est `00:00:01:04:2B:E0`, lancez la commande `scan off` pour arrêter la recherche d'appareils puis connectez-vous à l'enceinte :

```
trust 00:00:01:04:2B:E0
pair 00:00:01:04:2B:E0
connect 00:00:01:04:2B:E0
```

Votre enceinte devrait reconnaître la connexion, ça y est on a du son !

Pour éviter de répéter ce processus à chaque redémarrage du RPi, vous pouvez créer un script de connexion automatique :

```
mkdir -p ~/startup
nano ~/startup/pairbluetooth
```

Dans le fichier collez la configuration suivante en veillant bien à modifier l'adresse MAC par celle de votre enceinte :

```
#!/bin/bash
sudo systemctl stop snips-audio-server

# bluetooth
sudo bluetoothctl << EOF
connect 00:00:01:04:2B:E0
EOF

# restarting snips
sudo systemctl start snips-audio-server
```

Lancez la commande suivante pour rendre le script exécutable :

```
chmod +x ~/startup/pairbluetooth
```

Puis éditer le fichier ` ~/.bashrc ` via `nano ~/.bashrc ` en rajoutant à la fin :

```
wait
~/startup/pairbluetooth
```



### Installation du microphone ReSpeaker

Maintenant que nous avons du son en sortie, nous avons besoin d'un moyen de captation. 

Si vous avez choisi un ReSpeaker 4 mic array (ou un autre ReSpeaker), l'installation est très simple :

```
git clone https://github.com/respeaker/seeed-voicecard.git
cd seeed-voicecard
sudo ./install.sh
```

Puis lancez :

```
sudo raspi-config
```

Sélectionnez "**Advanced Options**", puis "**Audio**" puis "**Force 3.5mm ('headphone') jack**" et enfin "**Finish**". Faites ECHAP pour sortir.

On y est presque !

Sur votre ordinateur dans le terminal, lancez la configuration automatique via Sam :

```
sam setup audio
```

Suivez les instructions, s'il vous demande si vous disposez d'un Maker Kit, répondez non.

Cette procédure va générer sur votre RPi un fichier `asound.conf` qu'il va falloir éditer. Pour l'instant il doit ressembler à ça :

```
# The IPC key of dmix or dsnoop plugin must be unique
# If 555555 or 666666 is used by other processes, use another one

# use samplerate to resample as speexdsp resample is bad
defaults.pcm.rate_converter "samplerate"

pcm.!default {
    type asym
    playback.pcm "playback"
    capture.pcm "ac108"
}

pcm.playback {
    type plug
    slave.pcm "hw:ALSA"
}

# pcm.dmixed {
#     type dmix
#     slave.pcm "hw:0,0"
#     ipc_key 555555
# }

pcm.ac108 {
    type plug
    slave.pcm "hw:seeed4micvoicec"
}

# pcm.multiapps {
#     type dsnoop
#     ac108-slavepcm "hw:1,0"
#     ipc_key 666666
# }
```

Modifiez-le en lançant la commande `sudo nano /etc/asound.conf`, copiez-collez la configuration ci-dessous en remplaçant l'adresse MAC de votre enceinte bluetooth :

```
# The IPC key of dmix or dsnoop plugin must be unique
# If 555555 or 666666 is used by other processes, use another one

# use samplerate to resample as speexdsp resample is bad
defaults.pcm.rate_converter "samplerate"

pcm.!default {
    type asym
    playback.pcm "playback"
    capture.pcm "ac108"
}

pcm.playback {
    type plug
        slave.pcm {
            type bluealsa
            device "00:00:01:04:2B:E0"
            profile "a2dp"
        }
}

# pcm.dmixed {
#     type dmix
#     slave.pcm "hw:0,0"
#     ipc_key 555555
# }

pcm.ac108 {
    type plug
    slave.pcm "hw:seeed4micvoicec"
}

# pcm.multiapps {
#     type dsnoop
#     ac108-slavepcm "hw:1,0"
#     ipc_key 666666
# }
```

Enfin redémarrez le RPi avec la commande :

```
sudo reboot
```



## Configuration de Home Assistant

L'interface de configuration de Home Assistant est visible à cette adresse : [http://hassbian.local:8123](http://hassbian.local:8123/) (attention vous devez être connecté à votre réseau local pour pouvoir y accéder)

Home Assistant va vous demander de créer un compte utilisateur, choisissez un login et mot de passe.

La page "Aperçu" vous montre quels objets sont connectés à Home Assistant, pour l'instant la votre est bien vide, mais elle ne va pas le rester longtemps !

Cliquez sur l'onglet "**Configuration**" dans la barre latérale de gauche, puis "**Intégrations**". Ici vous pouvez cliquez sur le petit plus "+" jaune en bas à droite et rajouter des intégrations classiques.  Par exemple si vous avez un **Chromecast**, cliquez sur "Google Cast" et Home Assistant va tout configurer pour que vous puissiez le contrôler directement depuis l'interface.

Pour accéder à la liste complète des intégrations possibles, rendez vous sur [cette page](<https://www.home-assistant.io/components/>).

Certains composants ne sont malheureusement pas configurables aussi facilement, il va falloir aller gratter dans les fichiers de configurations pour les intégrer.





Configurer les devices dans el configuration.yaml

Rajouter le mqtt broker de snips 

127.0.0.1

1883




