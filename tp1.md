# Procédure de la séance 1 :

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

## 4) Créer et gérer des machines virtuelles

**Attention** vous devez exécuter toutes les commandes de cette section sur votre **machine de virtualisation**.


Nous vous fournissons un script nommé <span style="color:salmon">vmiut</span> qui vous permet de gérer vos machines virtuelles. Ce script n’est pas situé dans un chemin standard du système, vous ne pouvez donc pas l’exécuter directement.

Pour pouvoir l’utiliser, vous devez utiliser la commande suivante:
```
login@virt $ source /home/public/vm/vm.env
```
Puis, lancer la commande <span style="color:salmon">vmiut</span> sans paramètre affichera un message d’aide. Vous devrez utiliser la commande <span style="color:salmon">source</span> dans chaque nouveau shell.

### 4.1) Utilisation du script <span style="color:salmon">VMIUT</span>
Voici toutes les commandes basiques et utile pour manipuler des *machines virtuelles* : 

Pour créer une machine virtuelle, en remplaçant le <span style="color:salmon">nom</span> par le nom de votre machine virtuelle.
```
login@virt $ vmiut creer [nom]
 ```
Pour démarrer la machine virtuelle : 
```
login@virt $ vmiut demarrer [nom]
```
Pour arréter la machine virtuelle :
```
login@virt $ vmiut arreter [nom]
```
Pour supprimer la machine virtuelle : 
```
login@virt $ vmiut supprimer [nom]
```
Pour lister toutes les machines virtuelles disponible :
```
login@virt $ vmiut lister
```
Avoir des informations sur la machine :
```
login@virt $ vmiut info [nom]
```
Parmi les informations disponible, on peut voir par exemple l'adresse IP de la machine avec la ligne <span style="color:salmon">ip-possible</span>, si cette ligne est vide, il faut attendre un peu et relancer la commande.

## 5) La machine virtuelle

### 5.1) Informations sur le réseau et la VM

Afin de mieux comprendre comment fonctionne le *réseau virtuel principal*, voici quelques informations utiles :

|Machine       |Adresse       |
|---    |:-:    |
|Machine de virtualisation       |192.168.194.1       |
|Routeur, DNS       |192.168.194.2       |
|Adresses dynamiques (attribuées automatiquement)       |192.168.194.25-192.168.194.128       |

La machine virtuelle a été créée à partir d'un modèle. Voici les caractéristiques principales : 

- Distribution : Debian GNU/Linux 11(bullseye)
- Utilisateur standard:<span style="color:salmon">user</span>,mot de passe: <span style="color:salmon">user</span>
- Administrateur:<span style="color:salmon">root</span>, mot de passe: <span style="color:salmon">root</span>
- empreinte de clé SSH: <span style="color:salmon">SHA256:SUHhxVJVZFiBQ6/koNbZfA9reKHyzIrvPgJvOEJ8zuE</span>

### 5.2) Utilisation de la machine virtuelle

Pour pouvoir lancer la console virtuelle , il faut lancer la commande :
```
user@virt $ vmiut console matrix
```
On voit, que la commande ne marche pas, il faut utiliser une option spéciale de la commande SSH. Il faut donc au préalable <span style="color:salmon">se déconnecter</span>.

Puis ensuite afin d'utiliser la redirection graphique, utiliser la commande :
```
login@phys $ ssh -X virtu
```
Si on relance la commande <span style="color:salmon">console</span>, on peut désormais se connecter avec le compte <span style="color:salmon">user</span> pour continuer notre tp.

On peut maintenant à partir de la console se connecter à la *machine virtuelle* sur la *machine de virtualisation* en utilisant le SSH : 
```
login@virt $ ssh user@192.168.194.xx
```
En remplaçant xx par votre IP.

### 5.3) Changer la configuration réseau

Dans cette partie nous travaillerons depuis notre **console virtuelle**, en mode <span style="color:salmon">administrateur/root</span>.
Pour ce faire, il faudra se déconnecter de notre console, et la relancer en entrant les <span style="color:salmon">identifiants root</span> cette fois.

Une fois fait, il est préférable que notre machine ait toujours la même adresse IP, nous allons choisir l'adresse 192.168.194.3.

Premièrement, il faut couper l'interface réseau avec la commande :
```
root@vm # ifdown enp0s3
```

Ensuite, il faut modifier les fichiers <span style="color:salmon">/etc/network/interfaces</span> et <span style="color:salmon">/etc/resolv.conf</span> de façon à ce que la VM ait l'adresse statique 192.168.194.3 et qu'elle utilise le routeur 192.168.194.2 et serveur DNS 192.168.194.2.

pour le fichier <span style="color:salmon">interface</span> : 
```
allow-hotplug enp0s3
iface enp0s3 inet static
        address 192.168.194.3
        gateway 192.168.194.2
```

pour le fichier <span style="color:salmon">resolv.conf</span> :
```
domain localdomain
search localdomain
nameserver 192.168.194.2
```
à priori, le fichier <span style="color:salmon">resolv.conf</span> est déjà configuré comme au-dessus, mais on ne sait jamais.

On peut désormais redémarrer l'interface réseau avec la commande qui suit : 
```
root@vm # ifup enp0s3
```

On peut vérifier notre configuration en utilisant les commandes ci-dessous : 
- <span style="color:salmon">ip addr show</span>
- <span style="color:salmon">ip route show</span>
- <span style="color:salmon">host www.univ-lille.fr</span>

Utilisez la commande suivante pour redémarrer la machine virtuelle et vérifier que la configuration réseau est bien persistante
```
root@vm# reboot
```

## 6) Configurer et mettre à jour la machine virtuelle

### 6.1) Connexion <span style="color:salmon">root</span> et SSH

Afin de se connecter à la *machine virtuelle* en SSH en compte root, on peut essayer plusieurs alternatives, mais la plus intéressante reste d'utiliser la commande <span style="color:salmon">su</span>.

En couplant cette commande avec le paramètre --login, on peut se connecter facilement au compte *administrateur* à partir du compte *user*.
```
user@vm $ su --login
```
Cela devrait normalement vous passer en mode administrateur.

### 6.2) Accès extérieur pour les VM

La machine est connecté au *réseau virtuel principal*, il est privé et notre machine n'est pas routé. Autrement dit, aucune machine, autre que votre machine de virtualisation et vos machines virtuelles, n’a accès à ce réseau.

Afin de se connecter il faut configurer le <span style="color:salmon">proxy</span>, pour se faire il faut ajouter les variables d'environnements suivante :

- HTTP_PROXY=[proxy]
- HTTPS_PROXY=[proxy]
- http_proxy=[proxy]
- https_proxy=[proxy]
- NO_PROXY=localhost,[ip]

Ces variables d'environnements sont à ajouter dans le fichier <span style="color:salmon">/etc/environment</span> et en remplaçant proxy par <span style="color:salmon">http://cache.univ-lille.fr:3128</span> dans notre cas. Et la dernière pour NO_PROXY, l'adresse localhost.
Ce qui nous donne : 
```
HTTP_PROXY=http://cache.univ-lille.fr:3128
HTTPS_PROXY=http://cache.univ-lille.fr:3128
http_proxy=http://cache.univ-lille.fr:3128
https_proxy=http://cache.univ-lille.fr:3128
NO_PROXY=localhost,192.168.194.0/24,172.18.48.0/22
```

Pour tester si cela fonctionne, on peut essayer la commande <span style="color:salmon">wget</span> : 
```
user@vm $ wget https://www.framasoft.org
```




### 6.3) Mise à jour de la machine

Le modèle fourni est daté, il y a eu des mises à jours entre temps publiées par Debian, nous devons donc le mettre à niveau grâce à la commande <span style="color:salmon">apt</span> :
```
root@vm # apt update && apt full-upgrade
```

et laissez se terminer la mise à jour. Si cette dernière vous pose une question au sujet de <span style="color:salmon">GRUB</span>, cochez la case <span style="color:salmon">[ ] /dev/sda</span> à l’aide de la barre d’espacement.

### 6.4) Installer des outils

En utilisant <span style="color:salmon">apt</span>, on peut installer des outils plutôt utiles comme :

- vim
```
apt install vim
```
- less
```
apt install less
```
- tree
```
apt install tree
```
- rsync
```
apt install rsync
```

## 7) Quelques éléments additionnels

Afin d'aller plus vite durant la manipulation des différentes machines, on peut créer des scripts, en modifiant le fichier config qui se trouve à l'emplacement : <span style="color:salmon">$HOME/.ssh/config</span>

- créer un alias **virt** pour votre machine de virtualisation. Ainsi, si votre machine de virtualisation est meleze12.iutinfo.fr, alors utiliser la commande ssh virt effectura en réalité l’équivalent de la commande ssh meleze12.iutinfo.fr ;
```
Host virt
    HostName acajou13.iutinfo.fr
    ForwardAgent
```
- effectuer un transfert de votre agent SSH quand vous vous connectez sur la machine de virtualisation. Vous pourez ainsi diffuser votre clé publique sur votre VM et utiliser votre clé privé (via l’agent de votre machine physique) depuis la machine de virtualisation ;
```
Host virt
    HostName acajou13.iutinfo.fr
    ForwardAgent yes
```
- créer un alias **vm** pour votre machine virtuelle qui utilise par défaut l’utilisateur user et non votre login;
```
Host vm
    HostName 192.168.194.3
    User user
    ForwardAgent yes
```
- créer un alias **vmjump** pour pouvoir se connecter à votre VM directement depuis votre machine physique (directive ProxyJump). En d’autre terme, la commande ssh vmjump se connectera d’abord à la machine de virtualisation puis, automatiquement à la VM, sans autre manipulation de votre part.
```
Host vmjump
    HostName 192.168.194.3
    User user
    ForwardAgent yes
    ProxyJump virt
```