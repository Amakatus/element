# Procédure du tp5 :

## 1) Création de la nouvelle machine matrix

Tout d'abord il faut créer la machine matrix puis la démarer :

    login@virt $ vmiut creer matrix
    login@virt $ vmiut start matrix

## 2) Configuation de l'adresse IP

Il faut ensuite configurer l'adresse IP de la machine.

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

On peut maintenant à partir de la console se connecter à la *machine virtuelle* sur la *machine de virtualisation* en utilisant le SSH : 
```
login@virt $ ssh user@192.168.194.xx
```
En remplaçant xx par votre IP.

## 3) Configuration du proxy HTTP

La machine est connecté au *réseau virtuel principal*, il est privé et notre machine n'est pas routé. Autrement dit, aucune machine, autre que votre machine de virtualisation et vos machines virtuelles, n’a accès à ce réseau.

Afin de se connecter il faut configurer le <span style="color:salmon">proxy</span>, pour se faire il faut ajouter les variables d'environnements suivante en root (su -l):

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
## 4) Synchronisation du temps NTP

La machine physique et de virtualisation sont à l'heure, mais la machine virtuelle a une heure d'avance.

Si on affiche les événement systèmes de l'*unit service* <span style="color:salmon">systemd-timesyncd</span> avec le journalctl : 
```
user@vm # journalctl "_SYSTEMD_UNIT=systemd-timesyncd"
```

Cela ne nous affiche rien car elle la machine n'est pas synchronisée.

Afin de régler ce soucis il faut d'abord modifier le fichier <span style="color:salmon">/etc/systemd/timesyncd.conf </span>
```
root@vm # nano /etc/systemd/timesyncd.conf 
```
Puis dans ce fichier, rajouter la ligne <span style="color:salmon">ntp.univ-lille.fr</span> et ne pas oublier de **DECOMMENTER** la ligne.
```
NTP=ntp.univ-lille.fr
#FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org 3.debian.pool.ntp.org
#RootDistanceMaxSec=5
#PollIntervalMinSec=32
#PollIntervalMaxSec=2048
```

Si maintenant on utilise la commande <span style="color:salmon">systemctl</span> avec l'argument status, on doit obtenir, en supposant qu'on a reboot au préalable avec la commande:
```
root@vm # systemctl restart systemd-timesyncd
root@vm # systemctl status systemd-timesyncd
```
Affichage attendu
```
root@matrix:~# systemctl status systemd-timesyncd
● systemd-timesyncd.service - Network Time Synchronization
     Loaded: loaded (/lib/systemd/system/systemd-timesyncd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-11-24 10:03:37 CET; 59min left
       Docs: man:systemd-timesyncd.service(8)
   Main PID: 510 (systemd-timesyn)
     Status: "Initial synchronization to time server 193.49.225.86:123 (ntp.univ-lille.fr)."
      Tasks: 2 (limit: 1132)
     Memory: 1000.0K
        CPU: 35ms
     CGroup: /system.slice/systemd-timesyncd.service
             └─510 /lib/systemd/systemd-timesyncd

nov. 24 10:03:36 matrix systemd[1]: Starting Network Time Synchronization...
nov. 24 10:03:37 matrix systemd[1]: Started Network Time Synchronization.
nov. 24 09:03:37 matrix systemd-timesyncd[510]: Initial synchronization to time server 193.49.225.86:123 (ntp.univ-lille.fr).
```
Les trois dernières lignes, si on lance la commande avec les permissions <span style="color:salmon">user</span> ne seront pas affichées, mais on pourra quand même observer que notre service est **Active**.

## 5) Deploiement de vos clés SSH

Pour transmettre notre clé, il va falloir utiliser la commande <span style="color:salmon">ssh-copy-id</span>, qui est une commande faite pour ça. Il faut aussi rajouter l'argument <span style="color:salmon">-i</span>.

```
login@virt $ ssh-copy-id -i ~/.ssh/id_rsa.pub user@192.168.194.3
```

Si on essaye de se reconnecter en ssh, le client va nous demander de nouveau notre <span style="color:salmon">passphrase</span> pour déchiffrer le contenu du fichier <span style="color:salmon">id_rsa</span> en passant par un **agent SSH**.
Ensuite, lors d’utilisations successives pendant votre session, il n’aura plus à le faire. Vous pourrez donc vous connecter plusieurs fois sans avoir à saisir un mot de passe ou une passphrase.

## 6) Configuration de sudo

Le modèle fourni est daté, il y a eu des mises à jours entre temps publiées par Debian, nous devons donc le mettre à niveau grâce à la commande <span style="color:salmon">apt</span> :
```
root@vm # apt update && apt full-upgrade
```

et laissez se terminer la mise à jour. Si cette dernière vous pose une question au sujet de <span style="color:salmon">GRUB</span>, cochez la case <span style="color:salmon">[ ] /dev/sda</span> à l’aide de la barre d’espacement.

Pour installer sudo :
```
root@vm # apt install sudo
```

Pour donner les droits sudo à un user : 
```
root@vm # adduser user sudo
ou
root@vm # usermod -aG sudo nomutilisateur
```

Pour vérifier qu'on a bien les droits sudo en ayant au préalable **redémarrer** la machine: 
```
user@vm # sudo whoami
```

## 7) Configuration du nom d'hôte et du fichier de résolution DNS local

Pour changer de nom de machine il faut utiliser la commande <span style="color:salmon">hostname</span>.
```
root@vm # nano /etc/hostname
``` 

Et changer le nom par ce que l'on veut.

Il faut également modifier le fichier  <span style="color:salmon">/etc/hosts</span>,en remplaçant *debian* par *matrix* et en ajoutant les autres machines: 
```                              
127.0.0.1       localhost
127.0.1.1       matrix <- ici
192.168.194.4   rproxy
192.168.194.5   db
192.168.194.6   element
``` 

Afin de vérifier si la modification a bien eu lieu, on peut reboot notre machine.

<span style="font-weight: 800; font-size: 25px; font-style: normal;color : salmon">Maintenant répeter cette opération pour les machines db(192.168.194.5) et element(192.168.194.6). (N'oubliez pas de changer les addresses IP)</span>


--------------------------

# Sur la machine Matrix

## 1) Installation de Synapse

### 1.1) Installation du paquet sous Debian

Premièrement, il nous faut installer Synapse, pour se faire on se rend sur leur site et plus précisement dans le [guide d'installation sous Debian](https://matrix-org.github.io/synapse/latest/setup/installation.html#matrixorg-packages)

Il suffit de rentrer les 4 commandes suivantes : 

```
user@vm $ sudo -E apt install -y lsb-release wget apt-transport-https
user@vm $ sudo -E wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" |
    sudo tee /etc/apt/sources.list.d/matrix-org.list
user@vm $ sudo -E apt update
user@vm $ sudo -E apt install matrix-synapse-py3
```

Pour le nom du serveur il faut donc mettre virtu.iutinfo.fr:9090 (en remplaçant <span style="color: salmon"> virtu </span> par le nom de votre machine de virtualisation)

Le serveur écrira ses messages à destination de l’administrateur (les logs) dans le fichier <span style="color:salmon">/var/log/matrix-synapse/homeserver.log.virtualisation</span>.

### 1.2) Paramétrage spécifique pour une instance dans un réseau privé

Pour que notre serveur ne contacte aucun autre serveurs, il faut enlever les paramètres de la ligne <span style="color:salmon">trusted_key_servers</span> dans le fichier <span style="color:salmon">/etc/matrix-synapse/homeserver.yaml</span>.
```
user@vm $ sudo nano /etc/matrix-synapse/homeserver.yaml
Remplacer par trusted_key_servers: []
``` 

Ne pas oubliez de supprimer les serveurs <span style="color:salmon">déjà connus</span> auparavant (**ligne en dessous**).

---------------------

# Sur la machine DB

### 1) Pour installer <span style="color:salmon">Postgresql</span>

```
root@vm # apt install postgresql
```

### 1.2) s'assurer que le service est bien lancé

```
root@vm # systemctl status postgresql
```
### 1.3) Utilisation d'une base Postgres
[Doc utilisation postgres synapse](https://matrix-org.github.io/synapse/latest/postgres.html)

La bibliothèque C libpq permet aux programmes de communiquer avec le serveur de base de données PostgreSQL, elle est nécessaire au bon fonctionnement de postgres.
```
user@vm $ sudo apt install libpq5
``` 
Ensuite, on se place en tant qu'utilisateur postgres : 
``` 
su - postgres
```

On créer un nouvel utilisateur avec l'argument <span style="color:salmon"> --pwprompt</span>, pour pouvoir créer son mot de passe : 
``` 
createuser --pwprompt matrix
``` 
On recréer ensuite une base de données avec les arguments suivants :
- <span style="color:salmon"> --encoding</span> : Spécifiie l'encodage utilisé, ici **UTF-8**
- <span style="color:salmon"> --locale</span> : Spécifie l'endroit où doit être utilisé la base de données.
- <span style="color:salmon"> --template</span> : Spécifie la template a utilisé sur notre base de données, après recherche, on utilise **template0** sur des bases de données où les tables ne vont plus changer.
- <span style="color:salmon"> --owner</span> : Pour spécifier le propriétaire de la table, ici synapse_user que l'on vient de créer.
``` 
createdb --encoding=UTF8 --locale=C --template=template0 --owner=matrix matrix
``` 


Modifier les fichier /etc/postgresql/13/main/postgresql.conf et /etc/postgresql/13/main/pg_hba.conf

dans /etc/postgresql/13/main/postgresql.conf il faut décommenter Listen_adresses et le modifier

    listen_adresses = '*'

dans /etc/postgresql/13/main/pg_hba.conf il faut ajouter dans la partie IPV4 

    host  matrix  matrix 192.168.194.0/24  md5

Il faut restart postgresql

        root@vm $ systemctl restart postgresql


-----------------
# Sur la machine Matrix

Afin que notre serveur Synapse utilise par défaut une base de données <span style="color:salmon"> postgresql</span>, et non pas une base de données <span style="color:salmon"> sqlite</span>.
Il faut encore une fois modifier le fichier <span style="color:salmon">/etc/matrix-synapse/homeserver.yaml</span>, et changer la configuration du paramètre **database** comme ci-dessous : 

    database:
        name: psycopg2
        args:
            user: matrix
            password: matrix
            database: matrix
            host: 192.168.194.5
            cp_min: 5
            cp_max: 10

    suppr : database: /var/lib/matrix-synapse/homeserver.db


On **redémarre** synapse : 
```
user@vm $ sudo systemctl restart matrix-synapse
```



## sur rproxy


modifier /etc/nginx/sites-available/default: 

        server {
            listen 9090;
            listen [::]:9090;

            server_name _;

        location / {
            proxy_pass http://192.168.194.3:9090;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
        server {
            listen 9090;
            listen [::]:9090;

            server_name ayou21.iut-infobio.priv.univ-lille1.fr;

        location / {
            proxy_pass http://192.168.194.6:8080;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

## sur matrix

modifier /etc/matrix-synapse/homeserver.yaml :

    listeners:
        - port: 9090
        tls: false
        type: http
        x_forwarded: true
        bind_addresses: ['::1', '127.0.0.1','192.168.194.3']
        resources:
            - names: [client, federation]
            compress: false





## sur la machine physique ou de virtualisation modifier le .ssh/config :

    Host rproxy
        User user
        HostName 192.168.194.4
        LocalForward 0.0.0.0:9090 localhost:9090
        ForwardAgent yes
