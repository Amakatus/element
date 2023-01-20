---
title: Accès à un service HTTP sur la VM
author: Sacha BOUTON et Benoît MISPLON
---

## Prérequis pour effectuer la procédure : 
-   Avoir une [machine virtuelle basique fonctionnelle](./creation_vm.md)
-   Avoir connaissance du [vocabulaire technique](./introduction_et_vocabulaire.md)

Pour que le service que nous allons installer soit accessible de toutes les machines de TP de l’IUT, celui-ci doit écouter sur les interfaces réseaux de notre machine de virtualisation.

### 1.1) Un premier service pour tester

Afin d'installer le serveur HTTP `nginx` sur notre machine virtuelle il suffit de faire :
```
root@virt # apt install nginx
```

Pour vérifier que l'installation de `nginx` s'est bien effectuée : 
```
systemctl status nginx
```
On doit bien voir la ligne Active : `Active (running)`.

Pareil pour `curl`, il faut utiliser apt install : 
```
root@virt # apt install curl
```

Vérification d'accès au serveur `nginx` **depuis la VM** à l'aide de la commande : 
```
user@vm $ curl http://localhost
``` 

Pour joindre le serveur depuis la machine de virtualisation, on peut essayer de rajouter l'argument `-x `, pour faire une requête à l'ip de notre *machine virtuelle*: 

```
login@virt $ curl -x 192.168.194.3 http://localhost
```

Cela ne marchera probablement pas, car il faut **rajouter** le port `80` pour signifier que l'on essaye d'accéder à une page **HTTP !**
Ce qui nous donne : 

```
login@virt $ curl -x 192.168.194.3:80 http://localhost
```

### 1.2) Accès au service depuis la machine physique

Selon nous, cela n'est pas possible car nous sommes a **2** niveaux d'imbriquations : 

- `machine physique`: peut obtenir le service de la *machine de virtualisation*
- `machine de virtualisation`: peut obtenir le service de la *machine virtuelle*

On peut essayer d'utiliser la commande `curl` préalablement installé pour s'en rendre compte: 
```
login@phys $ curl -x acajou13.iutinfo.fr http://localhost
```

Cela ne marchera pas, on ne peut pas accèder de la **machine physique** à la **machine virtuelle**.

Pour résoudre ce problème : 

Grâce à  la commande `ssh` et l'argument `-L`, on se place dans notre machine virtuelle :

```
login@virt $ ssh -L :8080:localhost:80 user@192.168.194.3
```

Si on ouvre notre navigateur sur notre **machine physique** avec l'adresse `http://votre-machine-de-virtualisation.iutinfo.fr:8080/`, en remplaçant `machine de virtualisation` par la machine de virtualisation attribuée.

*Normalement, on obtient bien la page d'accueil de nginx !*

Si on a bien suivi la procédure [de configuration ssh](./configuration_ssh.md), normalement il n'y a rien a adapté pour pouvoir se passer de l'argument -L. (La ligne LocalForward sert justement à ça).

On peut donc passer à [l'installation de synapse](./installation_synapse.md).