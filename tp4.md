# Procédure 4
## Sacha Bouton et Benoît Misplon

## 1) Element Web

On a choisit apache : Pourquoi ? Et pourquoi pas nginx?

Nous avons donc choisi <span style='color:salmon'>*apache*</span> pour notre client Element
Au préalable, il faudra désinstaller Nginx ou changer le port car il y aura un conflit, en effet, les deux sont réglées sur le port <span style='color:salmon'>80</span>.
```
sudo -E apt install apache2
sudo apt autoremove nginx --purge -y -> désinstalle complétement nginx
``` 

Ensuite, afin d'activer notre <span style='color:salmon'>*serveur web*</span> : 

On se place dans le répertoire <span style='color:salmon'>/etc/apache2/sites-available</span> : 
```
user@vm $ cd /etc/apache2/sites-available
```
Puis, on <span style='color:salmon'>disable</span> les configurations par défauts avec la commande <span style='color:salmon'>a2dissite</span>.
```
user@vm:/etc/apache2/sites-available $ sudo a2dissite 000-default.conf
user@vm:/etc/apache2/sites-available $ sudo a2dissite default-ssl.conf
```
Ensuite, on fait un choix assez personnel de copier et de renommer le fichier qui va nous servir pour configurer notre serveur en <span style='color:salmon'>websites</span>, pour plus de **visibilité**.
```
user@vm:/etc/apache2/sites-available $ sudo cp 000-default.conf websites.conf
user@vm:/etc/apache2/sites-available $ sudo rm 000-default.conf
``` 

Et on <span style='color:salmon'>active</span> notre configuration avec la commande <span style='color:salmon'>a2ensite</span>.
```
user@vm:/etc/apache2/sites-available $ sudo a2ensite websites.conf
```
! <span style='color:salmon'>Important</span> :ne pas oublier de **redémarrer** notre serveur à chaque manipulation importante avec la commande 

```
user@vm $ sudo systemctl restart apache2 ou sudo service restart apache2
user@vm $ sudo systemctl reload apache2 ou sudo service reload apache2
```

Ensuite, il faut modifier le port afin qu'il écoute qu'Element soit accessible sur le <span style='color:salmon'>port 8080</span> dans le fichier <span style='color:salmon'>ports.conf</span>.
```
user@vm:/etc/apache2/ $ sudo nano ports.conf
Et remplacer par Listen 8080
```

Pareil dans le fichier websites.conf, on modifie pour avoir le <span style='color:salmon'>port 8080</span>.
```
user@vm:/etc/apache2/sites-available $ sudo nano websites.conf
Et mettre <VirtualHost *:8080>
```

Et pour finir, <span style='color:salmon'>**activer websites.conf et redémarrer apache2**</span>.
```
user@vm:/etc/apache2/sites-available $ sudo a2ensite websites.conf
user@vm:/etc/apache2/sites-available $ sudo systemctl restart apache2
``` 


Pour tester si notre serveur marche bien :
On peut modifier le fichier /var/www/html
Inserer image apache2

dl element:
https://github.com/vector-im/element-web

wget https://github.com/vector-im/element-web/releases/download/v1.11.16/element-v1.11.16.tar.gz
tar -xvzf element-v1.11.16.tar.gz 

Ensuite, config json faut le cp  


cd element-serv/
ls
cp config.sample.json config.json
ls
cd ..
ls
cp -R element-serv/ /var/www/
sudo cp -R element-serv/ /var/www/
cd /var/www/
ls
sudo rm -Rf html/
mv element-serv/ html
sudo mv element-serv/ html

## 2 Reverse proxy pour Synapse 

### 2.1) Introduction et choix d'un reverse proxy   

Recreer machine à l'aide de la procédure 1 , en changeant l'ip par 192.168.194.4 avec la gateawy sinon pas de réseau

-> Ajouter dans .ssh/config des scripts pour accéder rapidement à rproxy.


 procédure 2 
installer apt
installer sudo pour faciliter 
hostname rproxy



## Reverse Proxy (Benoît à modif) ##

modifier dans /etc/nginx/sites-available/default

    user@rproxy sudo nano /etc/nginx/sites-availables/default

    server {
        listen 80;
        listen [::]:80;

        server_name _;

    location / {
        proxy_pass http://192.168.194.3:8008;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    }

Modifier le proxy

    user@rproxy sudo nano /etc/environment

    HTTP_PROXY=http://cache.univ-lille.fr:3128
    HTTPS_PROXY=http://cache.univ-lille.fr:3128
    http_proxy=http://cache.univ-lille.fr:3128
    https_proxy=http://cache.univ-lille.fr:3128
    # NO_PROXY=localhost,192.168.194.0/24,172.18.48.0/22
    NO_PROXY=localhost,192.168.194.3,192.168.194.4

Modifier le /etc/matrix-synapse/homeserver.yaml

    user@matrix sudo nano /etc/matrix-synapse/homeserver.yaml

    bind_addresses: ['::1', '127.0.0.1', '192.168.194.3']

Restart nginx : 

    user@rproxy sudo systemctl restart nginx


Modifier .ssh/config

    nano .ssh/config

    Host rproxy
        HostName 192.168.194.4
        LocalForward 0.0.0.0:8008 localhost:80
        User user
        ForwardAgent yes

Pour tester sur la machine de virtualisation : 

    curl -x 192.168.194.4:80 localhost

>Renvoie le serveur matrix

Pour testet depuis rproxy :

    user@rproxy curl 192.168.194.3:8080
    user@rproxy curl 192.168.194.3:8008

>8080 doit renvoyer Element tandis que 8008 renvoie le serveur matrix.