# Procédure 3 
## Sacha Bouton et Benoît Misplon

## 1) Accès à un service HTTP sur la VM

### 1.1) Un premier service pour tester

Afin d'installer le serveur HTTP <span style="color:salmon">nginx</span> dans notre machine virtuelle il suffit de faire :
```
root@virt # apt install nginx
```

Pour vérifier que l'installation de <span style="color:salmon">nginx</span> s'est bien effectuée : 
```
systemctl status nginx
```
On doit bien voir la ligne Active a <span style="color:#4AF626">Active (running)</span>.

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


## 2) Installation de Synapse

### 2.1) Installation du paquet sous Debian

Premièrement, il nous faut installer Synapse, pour se faire on se rend sur leur site et plus précisement dans le [guide d'installation sous Debian](https://matrix-org.github.io/synapse/latest/setup/installation.html#matrixorg-packages)

Il suffit de rentrer les 4 commandes suivantes : 

```
user@vm $ sudo -E apt install -y lsb-release wget apt-transport-https
user@vm $ sudo - Ewget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" |
    sudo tee /etc/apt/sources.list.d/matrix-org.list
user@vm $ sudo - Eapt update
user@vm $ sudo - Eapt install matrix-synapse-py3
```

Le serveur écrira ses messages à destination de l’administrateur (les logs) dans le fichier <span style="color:salmon">/var/log/matrix-synapse/homeserver.log.virtualisation</span>.

### 2.2) Paramétrage spécifique pour une instance dans un réseau privé

Pour que notre serveur ne contacte aucun autre serveurs, il faut enlever les paramètres de la ligne <span style="color:salmon">trusted_key_servers</span> dans le fichier <span style="color:salmon">/etc/matrix-synapse/homeserver.yaml</span>.
```
user@vm $ sudo nano /etc/matrix-synapse/homeserver.yaml
Remplacer par trusted_key_servers: []
``` 

Ne pas oubliez de supprimer les serveurs <span style="color:salmon">déjà connus</span> auparavant (**ligne en dessous**).

### 2.3) Utilisation d'une base Postgres
[Doc utilisation postgres synapse](https://matrix-org.github.io/synapse/latest/postgres.html)

La bibliothèque C libpq permet aux programmes de communiquer avec le serveur de base de données PostgreSQL, elle est nécessaire au bon fonctionnement de postgres.
```
user@vm $ sudo apt install libpq5
``` 
Ensuite, on se place en tant qu'utilisateur postgres : 
``` 
su - postgres
``` 
On détruit notre base de données au préalable créer : 
``` 
dropdb matrix
``` 
On détruit également l'utilisateur qu'on avait créer avec : 
``` 
dropuser matrix
``` 
On créer un nouvel utilisateur avec l'argument <span style="color:salmon"> --pwprompt</span>, pour pouvoir créer son mot de passe : 
``` 
createuser --pwprompt synapse_user
``` 
On recréer ensuite une base de données avec les arguments suivants :
- <span style="color:salmon"> --encoding</span> : Spécifiie l'encodage utilisé, ici **UTF-8**
- <span style="color:salmon"> --locale</span> : Spécifie l'endroit où doit être utilisé la base de données.
- <span style="color:salmon"> --template</span> : Spécifie la template a utilisé sur notre base de données, après recherche, on utilise **template0** sur des bases de données où les tables ne vont plus changer.
- <span style="color:salmon"> --owner</span> : Pour spécifier le propriétaire de la table, ici synapse_user que l'on vient de créer.
``` 
createdb --encoding=UTF8 --locale=C --template=template0 --owner=synapse_user synapse
``` 

Afin que notre serveur Synapse utilise par défaut une base de données <span style="color:salmon"> postgresql</span>, et non pas une base de données <span style="color:salmon"> sqlite</span>.
Il faut encore une fois modifier le fichier <span style="color:salmon">/etc/matrix-synapse/homeserver.yaml</span>, et changer la configuration du paramètre **database** comme ci-dessous : 

    database:
        name: psycopg2
        args:
            user: synapse_user
            password: synapse
            database: synapse
            host: 127.0.0.1
            cp_min: 5
            cp_max: 10


On **redémarre** synapse : 
```
user@vm $ sudo systemctl restart matrix-synapse
```

Dans le fichier <span style="color:salmon">/etc/matrix-synapse/homeserver.yaml</span>, on rajoute aussi la ligne  :

    registration_shared_secret: "synapse"


### 2.4 ) Création d'utilisateur
Pour créer un utilisateur on utilise la commande <span style="color:salmon">sudo register_new_matrix_user</span> avec comme arguments : 
- <span style="color:salmon">-u</span> : Pour renseigner le nom d'utilisateur.
- <span style="color:salmon">-p</span> : Pour renseigner le mot de passe de l'utilisateur.
- <span style="color:salmon">-a</span> : Pour créer l'utilisateur en tant qu'**admin**.
- <span style="color:salmon">-c</span> : Le chemin vers le fichier contenant le *shared_secret*.
```
user@vm $ sudo register_new_matrix_user -u [nom] -p [password] -a -c /etc/matrix-synapse/homeserver.yaml
```

Si on veut créer 2 utilisateurs, on répète deux fois la commande , en changeant le nom d'utilisateur (on peut avoir le même mot de passe cependant)

### 2.5) Connexion à votre serveur Matrix 

Premièrement, on peut déjà ouvrir le site mis à disposition, [ici](http://tp.iutinfo.fr:8888/).

Avant de faire quoi que ce soit sur le site, il faut d'abord sortir de la *machine virtuelle* et se rendre sur notre *machine physique*, afin de modifier nos scripts créer précedemment dans la <span style="color:salmon">Procédure 1</span> : 

```
login@phy $ nano .ssh/config
````

Puis, dans le script VM, on rajoute les deux lignes suivantes  : 
```
LocalForward 0.0.0.0:8080 192.168.194.3:8080
LocalForward 0.0.0.0:8008 localhost:8008
```

On peut ensuite **revenir** sur notre *machine virtuelle*.

<span style="color:salmon">**Attention :**</span>
Il faut donc bien se connecter en utilisant le <span style="color:salmon">ssh vm</span>, sinon le script ne sera pas utilisé et le serveur ne sera pas opérationnel.


Se connecter et changer le serveur en mettant http://[nommachine].iutinfo.fr:8008

Et mettre les logins des utilisateurs qu'on a créer , pour ma part Sacha sacha.

Créer un salon et inviter l'autre utilisateur : 
Cliquer sur le bouton à droite créer un salon

Afin d'inviter on rentre :  @[nomutilisateur]:[machine].iutinfo.fr:8008
et on invite simplement.

## 2.6) Activation de l'enregistrement des utilisateurs


