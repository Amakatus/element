# Procédure 4
## Sacha Bouton et Benoît Misplon

## 1) Element Web

Nous avons choisi d'utiliser <span style='color:salmon'>Apache</span>, mais <span style='color:salmon'>Nginx</span> aurait pu être un choix viable puisque : 

- La principale différence entre les serveurs web NGINX et Apache et que NGINX a une architecture qui traite de multiples requêtes au sein d'un même thread, alors qu'Apache est piloté par les processsus et crée un thread pour chaque requête.

Chaque logiciel a ses avantages et ses inconvénients, de sorte que la décision d’utiliser NGINX ou Apache dépendra entièrement de nos préférences c'est pourquoi nous avons choisis <span style='color:salmon'>Apache</span> pour le serveur web :

Mis à part le multi-threading ce qui nous a aidé à choisir est que NGINX est plus rapide qu’Apache pour fournir du contenu statique, mais il a besoin de l’aide d’un autre logiciel pour traiter les demandes de contenu dynamique. D’autre part, Apache peut gérer le contenu dynamique en interne.

Dans notre cas le contenu est de type dynamique, le choix s'est donc porté sur <span style='color:salmon'>Apache</span>.

### 1.1) Configuration d'Apache
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


>Remarque : Pour tester si notre serveur marche bien,
On peut modifier le fichier <span style='color:salmon'>/var/www/html</span>, si les modifications sont bien prises en compte, c'est que tout est fonctionnel.  

### 1.2) Mise en place d'Element

Tout d'abord, il faut installer <span style='color:salmon'>Element</span>, pour se faire, il suffit d'utiliser la commande **wget** :
```
user@vm $ wget https://github.com/vector-im/element-web/releases/download/v1.11.16/element-v1.11.16.tar.gz
```

Cela nous donne une archive <span style='color:salmon'>tar.gz</span> qu'il faut désarchiver avec la commande **tar**
```
user@vm $ tar -xvzf element-v1.11.16.tar.gz 
```

Ensuite, il faut copier le <span style='color:salmon'>config.json</span>, en se plaçant dans le dossier désarchivé :

>Remarque : nous avons renomé le dossier <span style='color:salmon'>element</span> en **element-serv** au préalable avec la commande cp pour plus de lisibilité 
```
user@vm $ cp -R element-v1.11.16 element-serv
```
Puis :
```
user@vm:/element-serv $ cp config.sample.json config.json
user@vm: $ sudo cp -R element-serv/ /var/www/
user@vm:/var/www $ sudo rm -Rf html/
user@vm:/var/www $ sudo mv element-serv/ html
```

>Après avoir fait les commandes ci-dessus, on obtient un element implanté dans apache et est **accessible** via le <span style='color:salmon'>serveur web</span>. 

## 2 Reverse proxy pour Synapse 

### 2.1) Introduction et choix d'un reverse proxy   

Pour le <span style='color:salmon'> reverse proxy, nous avons choisi d'utiliser nginx</span> :
Au lieu de choisir soit NGINX soit Apache, il peut être plus efficace d’utiliser les deux logiciels pour améliorer les performances de votre serveur – NGINX comme serveur proxy inverse pour traiter les demandes de contenu statique et Apache comme back-end pour servir le contenu dynamique.

Il faut tout d'abord <span style='color:salmon'>recréer</span> une machine de zéro, pour cela il nous faudra suivre les procédures 1 2 et 3, voici les choses importantes :

- 1- Créer une machine avec **vmiut** (<span style='color:salmon'>Procédure 1</span>)
- 2- Configurer le réseau de la machine en utilisant comme **IP** : 192.168.194.4 et ne pas oublier la **gateway** (<span style='color:salmon'>Procédure 1</span>)
- 3- Mise à jour de la machine avec **apt** (<span style='color:salmon'>Procédure 1</span>)
- 4- Installer **sudo** pour faciliter l'utilisation de la machine (<span style='color:salmon'>Procédure 2</span>)
- 5- Changer l'hostname de la machine en **rproxy** (<span style='color:salmon'>Procédure 2</span>)

-> Ajouter dans .ssh/config des scripts pour accéder rapidement à rproxy.

## Reverse Proxy (Benoît à modif) ##

Nous utiliserons Nginx pour le reverse proxy [expliquer]

Premièrement, il faut l'installer avec **apt** :
```
user@vm $ sudo -E apt install nginx
```
Ensuite, il faut aller modifier les **sites-available**, afin qu'il écoute bien sur le port *80*, il faut aussi modifier le <span style='color:salmon'>proxy_pass</span> afin que notre reverse proxy renvoie bien à notre machine virtuelle **matrix** qui elle possède le *serveur web* et <span style='color:salmon'>**Element**</span>.
```
user@rproxy sudo nano /etc/nginx/sites-availables/default
```
Notre configuration par défaut devrait ressembler à ça après modification :
```
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
```
Il faut aussi modifier les <span style='color:salmon'>variables d'environnements</span> et renseigner un **NO_PROXY** avec les **ip** de <span style='color:salmon'>matrix</span> et celle du <span style='color:salmon'>rproxy</span>.
```
user@rproxy sudo nano /etc/environment
```
Rajouter le **NO_PROXY** en n'oubliant pas de **commenter** la ligne au dessus sinon le proxy ne sera pas effectif.
```
HTTP_PROXY=http://cache.univ-lille.fr:3128
HTTPS_PROXY=http://cache.univ-lille.fr:3128
http_proxy=http://cache.univ-lille.fr:3128
https_proxy=http://cache.univ-lille.fr:3128
# NO_PROXY=localhost,192.168.194.0/24,172.18.48.0/22
NO_PROXY=localhost,192.168.194.3,192.168.194.4
```

De plus, sur la machine *machine virtuelle* <span style='color:salmon'>matrix</span> afin de changer la configuration du fichier <span style='color:salmon'>homeserver.yaml</span>.

```
user@matrix sudo nano /etc/matrix-synapse/homeserver.yaml
bind_addresses: ['::1', '127.0.0.1', '192.168.194.3']
```

>Remarque : Ne pas oublier de redémarrer le service nginx ainsi que le service matrix-synapse après les manipulations.
```
user@rproxy $ sudo systemctl restart nginx
user@matrix $ sudo systemctl restart matrix-synapse
```

Notre configuration de nginx est terminée.
**Cependant**, il reste encore à configurer notre configuration ssh et le tunnel pour <span style='color:salmon'>rproxy</span>.
```
login@phy $ nano .ssh/config
```

```
Host rproxy
    HostName 192.168.194.4
    LocalForward 0.0.0.0:8008 localhost:80
    User user
    ForwardAgent yes
```

Afin de voir si nos configurations sont correctes on peut effectuer divers tests :

Pour tester sur la *machine de virtualisation* : 
```
login@virt $ curl -x 192.168.194.4:80 localhost
```
>Doit renvoyer le serveur matrix.

Pour tester depuis *rproxy* :
```
user@rproxy $ curl 192.168.194.3:8080
user@rproxy $ curl 192.168.194.3:8008
```
>8080 doit renvoyer Element.
 
>8008 renvoie le serveur matrix.