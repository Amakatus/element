# Procédure 3 
## Sacha Bouton et Benoît Misplon

## 1) Accès à un service HTTP sur la VM

### 1.1) Un premier service pour tester

Afin d'installer le serveur HTTP <span style="color:salmon">nginx</span> dans notre machine virtuelle il suffit de faire :
```
root@virt # apt install nginx
```

Pour vérifier l'installation de <span style="color:salmon">nginx</span>
```
systemctl status nginx
```
On doit bien voir la ligne Active a <span style="color:#4AF626">Active (running)</span> 

Pareil pour <span style="color:salmon">curl</span>, il faut utiliser apt install : 
```
root@virt # apt install curl
```

Vérification d'accès au serveur <span style="color:#4AF626">nginx</span> **depuis la VM** à l'aide de la commande : 
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

Selon nous, cela n'est pas possible car nous sommes a 2 niveau d'imbriquation : 
```
physique -> peut obtenir le service de la machine de virtualisation
virtualisation -> peut obtenir le service de la machine virtuelle
```

On peut essayer la commande :
```
login@phys $ curl -x acajou13.iutinfo.fr http://localhost
```

Pour résoudre ce problème : 

Grâce à  la commande ssh et l'argument <span style="color:salmon">-L</span>, on se place dans notre machine virtuelle :

```
login@virt $ ssh -L :8080:localhost:80 user@192.168.194.3
```

Si on ouvre notre navigateur sur notre **machine physique** avec l'adresse <span style="color:salmon">http://votre-machine-de-virtualisation.iutinfo.fr:8080/</span>, en remplaçant <span style="color:salmon">machine de virtualisation</span> par la machine de virtualisation attribuée.


## 2) Installation de Synapse

### 2.1) Installation du paquet sous Debian

Afin d'installer Synapse on se rend sur le site et on prends les commandes suivantes : 
```
sudo apt install -y lsb-release wget apt-transport-https
sudo wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" |
    sudo tee /etc/apt/sources.list.d/matrix-org.list
sudo apt update
sudo apt install matrix-synapse-py3
```

Le serveur écrira ses messages à destination de l’administrateur (les logs) dans le fichier <span style="color:salmon">/var/log/matrix-synapse/homeserver.log.virtualisation</span>.

### 2.2 Paramétrage spécifique pour une instance dans un réseau privé
[Doc utilisation postgres synapse](https://matrix-org.github.io/synapse/latest/postgres.html)


    user@vm sudo apt install libpq5
    su - postgres
    dropdb matrix
    dropuser matrix
    createuser --pwprompt synapse_user
    createdb --encoding=UTF8 --locale=C --template=template0 --owner=synapse_user synapse


Modifier la configuration synapse afin d'utiliser postgres en modifiant le fichier <span>/etc/matrix-synapse/homeserver.yaml</span> et y modifier la partie <span>database</span>

    database:
        name: psycopg2
        args:
            user: synapse_user
            password: synapse
            database: synapse
            host: 127.0.0.1
            cp_min: 5
            cp_max: 10


On redémarre synapse avec `user@vm sudo systemctl restart matrix-synapse`

Dans le fichier `/etc/matrix-synapse/homeserver.yaml` :

    registration_shared_secret: "synapse"

### 2.4 ) Création d'utilisateur
Puis on créer un utilisateur : 

    sudo register_new_matrix_user -u Benoit -p benoit -a -c /etc/matrix-synapse/homeserver.yaml


### 2.5)

nano .ssh/config
Rajouter à vm 
        LocalForward 0.0.0.0:8080 192.168.194.3:8080
        LocalForward 0.0.0.0:8008 localhost:8008

Utiliser les scritps pour se mettre sur la machine virtuelle avec le script VM.

Se rendre sur l'adresse   http://tp.iutinfo.fr:8888/.

Se connecter et changer le serveur en mettant http://[nommachine].iutinfo.fr:8008

Et mettre les logins des utilisateurs qu'on a créer , pour ma part Sacha sacha.

Créer un salon et inviter l'autre utilisateur : 
Cliquer sur le bouton à droite créer un salon

Afin d'inviter on rentre :  @[nomutilisateur]:[machine].iutinfo.fr:8008
et on invite simplement.


