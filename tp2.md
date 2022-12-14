# Procédure 2 

## 1) Dernière configuration sur la VM

### 1.1) Changement du nom de machine

Pour changer de nom de machine il faut utiliser la commande <span style="color:salmon">hostname</span>.
```
root@vm # nano /etc/hostname
``` 

Et changer le nom par ce que l'on veut.

Il faut également modifier le fichier  <span style="color:salmon">/etc/hosts</span>,en remplaçant *debian* par *matrix* : 
```                              
127.0.0.1       localhost
127.0.1.1       matrix <- ici
``` 

Afin de vérifier si la modification a bien eu lieu, on peut reboot notre machine.

### 1.2) Installation et configuration de la commande <span style="color:salmon">sudo</span>

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

### 1.3) Configuration de la synchronisation d'horloge
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


## 2 Installation et configuration basique d'un serveur de base de données

### 2.1) Pour installer <span style="color:salmon">Postgresql</span>

```
root@vm # apt install postgresql
```

### 2.2) s'assurer que le service est bien lancé

```
root@vm # systemctl status postgresql
```
### 2.3) Se connecter à la base de données <span style="color:salmon">Postgresql</span>

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



### 2.4) Créer une base de données et la manipuler 

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



