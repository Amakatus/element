---
title: Accès à un service HTTP sur la VM
author: Sacha BOUTON et Benoît MISPLON
---

## Conditions pour effectuer la procédure : 
-   Avoir une [machine virtuelle basique fonctionnelle](./creation_vm.md)
-   Avoir connaissance du [vocabulaire technique](./introduction_et_vocabulaire.md)

Pour que le service que nous allons installer soit accessible de toutes les machines de TP de l’IUT, celui-ci doit écouter sur les interfaces réseaux de notre machine de virtualisation.

### 1.1) Un premier service pour tester

Afin d'installer le serveur HTTP <span style="color:salmon">nginx</span> sur notre machine virtuelle il suffit de faire :
```
root@virt # apt install nginx
```

Pour vérifier que l'installation de <span style="color:salmon">nginx</span> s'est bien effectuée : 
```
systemctl status nginx
```
On doit bien voir la ligne Active : <span style="color:#4AF626">Active (running)</span>.

Pareil pour <span style="color:salmon">curl</span>, il faut utiliser apt install : 
```
root@virt # apt install curl
```

Vérification d'accès au serveur <span style="color:salmon">nginx</span> **depuis la VM** à l'aide de la commande : 
```
user@vm $ curl http://localhost
``` 

Pour joindre le serveur depuis la machine de virtualisation, on peut essayer de rajouter l'argument <span style="color:salmon">-x </span>, pour faire une requête à l'ip de notre *machine virtuelle*: 

```
login@virt $ curl -x 192.168.194.3 http://localhost
```

Cela ne marchera probablement pas, car il faut **rajouter** le port <span style="color:salmon">80</span> pour signifier que l'on essaye d'accéder à une page **HTTP !**
Ce qui nous donne : 

```
login@virt $ curl -x 192.168.194.3:80 http://localhost
```

### 1.2) Accès au service depuis la machine physique

Selon nous, cela n'est pas possible car nous sommes a **2** niveaux d'imbriquations : 

- <span style="color:salmon">machine physique</span>: peut obtenir le service de la *machine de virtualisation*
- <span style="color:salmon">machine de virtualisation</span>: peut obtenir le service de la *machine virtuelle*

On peut essayer d'utiliser la commande <span style="color:salmon">curl</span> préalablement installé pour s'en rendre compte: 
```
login@phys $ curl -x acajou13.iutinfo.fr http://localhost
```

Cela ne marchera pas, on ne peut pas accèder de la **machine physique** à la **machine virtuelle**.

Pour résoudre ce problème : 

Grâce à  la commande <span style="color:salmon">ssh</span> et l'argument <span style="color:salmon">-L</span>, on se place dans notre machine virtuelle :

```
login@virt $ ssh -L :8080:localhost:80 user@192.168.194.3
```

Si on ouvre notre navigateur sur notre **machine physique** avec l'adresse <span style="color:salmon">http://votre-machine-de-virtualisation.iutinfo.fr:8080/</span>, en remplaçant <span style="color:salmon">machine de virtualisation</span> par la machine de virtualisation attribuée.

*Normalement, on obtient bien la page d'accueil de nginx !*

Si on a bien suivi la procédure [de configuration ssh](./configuration_ssh.md), normalement il n'y a rien a adapté pour pouvoir se passer de l'argument -L. (La ligne LocalForward sert justement à ça).

On peut donc passer à [l'installation de synapse](./installation_synapse.md).