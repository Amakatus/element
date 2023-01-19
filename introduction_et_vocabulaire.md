---
title: Introduction et vocabulaire
author: Sacha BOUTON et Benoît MISPLON
---

## Conditions pour effectuer la procédure : 
-   Aucune

## 1) Un peu de vocabulaire et de convention

Afin de bien se comprendre tout au long de cette SAE, voici le vocabulaire de base qui sera employé durant les procédures :

- **machine physique :**  la machine qui est devant vous. Celle sur laquelle vous êtes connecté.e.s dans la salle TP ;

- **machine de virtualisation :** la machine qui fera fonctionner vos machines virtuelles. Vous vous y connecterez à distance depuis la *machine physique*. Cette machine vous sera attribuée et sera la même tout au long de la SAÉ. Par un hasard de l’emploi du temps, il se pourrait qu’elle soit parfois la même que la *machine physique*, mais c’est peu probable ;

- **machine virtuelle :** une machine “émulée” par un logiciel de virtualisation. Les *machines virtuelles* seront accessibles via la machine de virtualisation ;

- **réseau physique :** il s’agit du réseau des salles de TP de l’IUT. Les *machines physiques* et de *virtualisation* y sont connectées directement via leur interface réseau <span style="color:salmon">eth0</span> ;

- **réseau virtuel principal :** il s’agit d’un réseau virtuel, reliant la *machine de virtualisation* aux *machines virtuelles* s’y exécutant. Sur la *machine de virtualisation*, ce réseau correspond à l’interface <span style="color:salmon">vmnet8</span>. Au début de la SAÉ, toutes vos machines virtuelles seront connectées à ce réseau. En fin de SAÉ, nous mettrons en place une architecture de réseau plus complexe. **Attention** chaque machine de virtualisation possède son **propre** réseau virtuel principal. En d’autres termes, deux machines virtuelles exécutées sur deux machines de virtualisation différentes ne seront pas concrètement sur le même réseau !

Pendant cette SAÉ, les sujets contiendront souvent des commandes à exécuter, avec parfois le résultat attendu de la commande, comme par exemple:

```
$ ls
README.html       README.md         network-sae.svg   sujet1.html       sujet1.md
```
Le prompt de votre invite de commande est indiqué par le caractère <span style="color:salmon">\$</span>, il ne faudra donc pas le saisir. Le prompt pourra également être indiqué par le caractère <span style="color:salmon">#</span> à la place de <span style="color:salmon">$</span>. Dans ce cas, il faudra exécuter la commande en tant qu’utilisateur root. Par exemple:
```
# adduser toto
```
Vous ne pourrez donc éxécuter ces commandes que sur vos *machines virtuelles*.

Si nécessaire, le prompt indiquera également précisemment l’utilisateur et la machine sur laquelle la commande doit s’exécuter sous la forme utilisateur@machine. Concernant la machine, il s’agira soit:

-  de <span style="color:salmon">phys</span> pour représenter la machine physique;
-  de <span style="color:salmon">virtu</span> pour représenter la machine virtuelle;
-  de <span style="color:salmon">vm</span> pour représenter la machine virtuelle dans le cas où il n'y a pas d'ambiguïté;
-  d'un nom de machine virtuelle s'il est nécessaire de le préciser

Dans le cas du nom d’utilisateur, le seul cas particulier sera <span style="color:salmon">login</span>. Ce cas se produira pour les commandes à saisir sur la machine physique ou la machine de virtualisation et il voudra simplement dire que vous devez exécuter la commande en utilisant votre compte étudiant.

Par exemple, si la commande <span style="color:salmon">ls</span> est à exécuter sur la machine physique:
```
login@phys$ ls
```
Si la commande <span style="color:salmon">id</span> est à éxécuter en tant qu'utilisateur <span style="color:salmon">root</span> de la machine virtuelle nommée <span style="color:salmon">matrix</span>:
```
root@matrix# id
```
La convention <span style="color:salmon">utilisateur@machine</span> pourra également être utilisée pour désigner les paramètres de connexion de la commande <span style="color:salmon">ssh</span>.Les mêmes noms particuliers s'appliqueront.

## 2) Première connexion à la machine de virtualisation
Afin de vous connecter à la machine de virtualisation utilisez la commande suivante en remplaçant <span style="color:salmon">machine</span> par le nom de la machine qui vous a été attribuée:
```
login@phys$ ssh machine.iutinfo.fr
```
Une fois la commande lancée, si vous ne vous étiez jamais connecté en SSH sur la machine indiquée, vous aurez un affichage similaire à celui-ci, ne validez rien tant que vous n’aurez pas effectué les vérifications indiquées après:
```
The authenticity of host 'teck03.iutinfo.fr (172.18.48.223)' can't be established.
ED25519 key fingerprint is SHA256:sefLeWVX2+oaIsGPjmgMhGfaQNF6UmGOAxJD4TBGnEI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
Ce message indique que le client SSH ne peut vérifier l'identité du serveur SSH. Il vous demande de faire **confiance** à ce serveur.

Il faut donc vérifier **l'empreinte** de sa clé. Une liste qui associe chaque machine de TP de l'IUT avec son empreinte est disponible sur moodle.

De mon côté, j'utilise la machine acajou13, la clé associée a cette machine est :
```
256 SHA256:ppyT9ZYYLTPzkVyZlDwqV80MZWU+brksx0OIABEee10
```

Après avoir vérifié l'empreinte, on peut répondre <span style="color:salmon">yes</span> à la question pour se connecter au client SSH, qui ajoutera le serveur dans le fichier <span style="color:salmon">$HOME/.ssh/known_hosts</span> pour indiquer que vous lui faites confiance.

## 3) Faciliter la connexion SSH

### 3.1) Fabriquer sa paire de clés
Devoir saisir son mot de passe à chaque connexion peut vite s’avérer pénible, surtout si on doit le faire souvent. SSH permet de s’authentifier autrement qu’avec un simple mot de passe.

En utilisant un mécanisme de cryptographie : **une paire de clés**

Il va donc falloir :
- fabriquer une paire de clés;
- transmettre la clé publique au serveur.

Pour fabriquer une paire de clé, vous allez utiliser la commande <span style="color:salmon">ssh-keygen</span> avec les paramètres par défaut2. Lors de l’utilisation, la commande vous demandera deux choses:
- un nom de fichier : vous pouvez laisser le nom de fichier par défaut. Notez le. Il est bon de le connaître ;
- une passphrase: c’est un mot de passe qui permet de chiffrer le fichier contenant votre clé privé. Il est très important d’utiliser un mot de passe pertinent. Ainsi, si on vous vole le fichier, le voleur ne pourra pas se servir de votre clé.

Voici un exemple d’exécution de la commande:
```
login@phys$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/infoetu/login/.ssh/id_rsa):
Created directory '/home/infoetu/login/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/infoetu/login/.ssh/id_rsa
Your public key has been saved in /home/infoetu/login/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:fMB2KbRIhY0++6x3VcQ8Zmzz9taiJdE79RREr7oPlLw login@phys
The key's randomart image is:
+---[RSA 3072]----+
|      .=o    +oo |
|     .o+.. .  @..|
|     .. * o  * +o|
|      oo + ...o.=|
|       oS . +o.=+|
|      .  . .oo= =|
|       o   .E+ + |
|        + . .o   |
|      .o .  ...  |
+----[SHA256]-----+
```

Par exemple, de mon côté le nom du fichier est stocké dans <span style="color:salmon">/home/infoetu/sacha.bouton.etu/.ssh/id_rsa.pub</span>. Nous avons donc créer notre paire de clés, il ne reste plus qu'à informer le serveur.

### 3.2) Transmettre sa paire de clés
Nous allons devoir transmettre la paire de clés <span style="color:salmon">id_rsa</span> qui se trouve sur notre *machine physique* au fichier où sont recensées les clés autorisées qui se trouve <span style="color:salmon">$HOME/.ssh/authorized_keys</span> de la *machine de virtualisation*.

Pour transmettre notre clé, il va falloir utiliser la commande <span style="color:salmon">ssh-copy-id</span>, qui est une commande faite pour ça. Il faut aussi rajouter l'argument <span style="color:salmon">-i</span>, qui va nous permettre de spécifier le chemin vers la paire de clés de notre *machine physique*.

Exemple du squelette de la commande :
```
login@phy $ ssh-copy-id -i [path] user@virtu
```

En d'autres termes, en prenant exemple avec ma configuration et mon emplacement de fichier créer au préalable.
```
login@phy $ ssh-copy-id -i /home/infoetu/sacha.bouton.etu/.ssh/id_rsa.pub sacha.bouton.etu@acajou13
```

Note :
On pourra faire pareil sur la *machine virtuelle* pour notre future machine virtuelle.
```
login@virt $ ssh-copy-id -i ~/.ssh/id_rsa.pub user@192.168.194.3
```

Si on essaye de se reconnecter en ssh, le client va nous demander de nouveau notre <span style="color:salmon">passphrase</span> pour déchiffrer le contenu du fichier <span style="color:salmon">id_rsa</span> en passant par un **agent SSH**.
Ensuite, lors d’utilisations successives pendant votre session, il n’aura plus à le faire. Vous pourrez donc vous connecter plusieurs fois sans avoir à saisir un mot de passe ou une passphrase.

Vous pouvez maintenant passer à la procédure pour [creer une machine virtuelle](./creation_vm.md).