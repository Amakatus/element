Conditions : 



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