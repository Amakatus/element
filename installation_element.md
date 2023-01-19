---
title: Installation d'element
author: Sacha BOUTON et Benoît MISPLON
---

## Conditions pour effectuer la procédure : 
-   Avoir une [machine virtuelle basique fonctionnelle](./creation_vm.md).
-   Avoir connaissance du [vocabulaire technique](./introduction_et_vocabulaire.md).
-   Avoir une [base de données basique fonctionnelle](./installation_bdd.md).
-   Avoir effectué le [premier accès au service http](./acces_au_service_http.md) pour comprendre le fonctionnement.
-   Avoir installé [synapse correctement](./installation_synapse.md)

**Element** est un logiciel libre de messagerie instantanée qui implémente Matrix. Pour l'utiliser nous avons besoin d'un **serveur web** : 
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

Une fois cela fait, notre serveur web est correctement mis en place et on peut passer au déploiement du [reverse proxy](./installation_reverse_proxy.md).