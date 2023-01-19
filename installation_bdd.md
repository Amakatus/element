---
title: Installation et configuration basique d'un serveur de base de données
author: Sacha BOUTON et Benoît MISPLON
---

Il faut désormais installer **Postgresql** qui va nous servir à mettre en place notre base de données.

### 1) Pour installer <span style="color:salmon">Postgresql</span>

Toujours sur matrix, installer **postgresql**

```
root@vm # apt install postgresql
```

Pour s'assurer que le service est bien lancé

```
root@vm # systemctl status postgresql
```
### 2) Se connecter à la base de données <span style="color:salmon">Postgresql</span>

Il faut au préalable se connecter à la base de données :
```
user@vm # sudo -u postgres -i
```

Puis créer l'utilisateur : 
```
createuser -r -l -P -d matrix
```
- r : Ajoute directement comme un nouveau membre. 
- l : Pour se login.
- P : Pour obliger d'avoir un mot de passe 
- d : Le droit de créer une base de donnée.



### 3) Créer une base de données et la manipuler 

Afin de créer une nouvelle base de données qui a comme <span style="color:salmon">owner</span> :  matrix et comme <span style="color:salmon">nom</span> : matrix

```
createdb -O matrix matrix
```

Afin de se connecter à la base de données que l'on vient de créer il faut utiliser (comme en TP de BDD): 
```
psql -h localhost -U matrix -d matrix
```

Ensuite, on créée la <span style="color:salmon">table</span> *test* dans la base de données matrix: 
```
Create table test(personid int ,prenomvar char(25));
```
Puis on utilise des lignes SQL basiques afin de tester si cela fonctionne bien : 
- Un insert
```
Insert into test (personid,prenom) values(10,'Sacha');
```
- Puis un select : 
```
Select * from test;
Select prenom from test where prenom = 'Sacha';
```

La base de données est donc bien installé pour une utilisation basique, qui nous servira pour la configuration basique, on peut donc passer à [l'acces au service http](./acces_au_service_http.md)