---
title: Installation du reverse proxy
author: Sacha BOUTON et Benoît MISPLON
---

## Conditions pour effectuer la procédure : 
-   Avoir une [machine virtuelle basique fonctionnelle](./creation_vm.md).
-   Avoir connaissance du [vocabulaire technique](./introduction_et_vocabulaire.md).
-   Avoir une [base de données basique fonctionnelle](./installation_bdd.md).
-   Avoir effectué le [premier accès au service http](./acces_au_service_http.md) pour comprendre le fonctionnement.
-   Avoir installé [synapse correctement](./installation_synapse.md)
-   Avoir installé [element](./installation_element.md)

## 1) Installation et choix :

Un reverse proxy est utile pour permettre à un utilisateur d'accéder à des serveurs internes au réseau, dans notre cas une requête sur un port spécifique permettra à l'utilisateur d'accéder à notre serveur Element, où à matrix.

Rappelons que nous avons utilisé `Apache` pour notre serveur Element Web.

Il peut être plus efficace d’utiliser les deux logiciels pour améliorer les performances de votre serveur : 
`NGINX` comme **serveur proxy inverse** pour traiter les demandes de contenu statique et Apache comme back-end pour servir le contenu dynamique.

C'est pour quoi comme reverse proxy, `NGINX` peut être plus pertinent.

Il faut au préalable [créer une nouvelle machine virtuelle](./creation_vm.md) avec comme configuration :
 - nom: `rproxy`
 - adresse ip : 192.168.194.4
 - gateway : 192.168.194.2

Premièrement, il faut l'installer avec **apt** :
```
user@vm $ sudo -E apt install nginx
```
Ensuite, il faut aller modifier les **sites-available**, afin qu'il écoute bien sur le port *80*, il faut aussi modifier le `proxy_pass` afin que notre reverse proxy renvoie bien à notre machine virtuelle **matrix** qui elle possède le *serveur web* et `**Element**`.
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
Il faut aussi modifier les `variables d'environnements` et renseigner un **NO_PROXY** avec les **ip** de `matrix` et celle du `rproxy`.
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

De plus, sur la machine *machine virtuelle* `matrix` afin de changer la configuration du fichier `homeserver.yaml`.

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
**Cependant**, il reste encore à configurer notre configuration ssh et le tunnel pour `rproxy`.
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

Cela facilitera la connexion à `rproxy` en plus de faire un tunnel pour ne pas devoir utiliser l'argument *-L*.
## 2) Tests de fonctionnalitées : 

Afin de voir si nos configurations sont **correctes** on peut effectuer divers tests :

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

Tous les éléments sont donc installer sur nos deux machines, il est donc tant de passer à [l'architecture finale](./architecture_finale.md)