---
title: Configuration SSH des machines
author: Sacha BOUTON et Benoît MISPLON
---

## Conditions pour effectuer la procédure : 
-   Avoir une [machine virtuelle basique fonctionnelle](./creation_vm.md)
-   Avoir connaissance du [vocabulaire technique](./introduction_et_vocabulaire.md)


Afin d'aller plus vite durant la manipulation des différentes machines, on peut créer des scripts, en modifiant le fichier config qui se trouve à l'emplacement : `$HOME/.ssh/config`

- créer un alias **virt** pour notre machine de virtualisation. Ainsi, si notre machine de virtualisation est meleze12.iutinfo.fr, alors utiliser la commande ssh virt effectura en réalité l’équivalent de la commande ssh meleze12.iutinfo.fr :
```
Host virt
    HostName acajou13.iutinfo.fr
    ForwardAgent
```
- effectuer un transfert de notre agent SSH quand vous vous connectez sur la machine de virtualisation. Vous pourez ainsi diffuser notre clé publique sur notre VM et utiliser notre clé privé (via l’agent de notre machine physique) depuis la machine de virtualisation :
```
Host virt
    HostName acajou13.iutinfo.fr
    ForwardAgent yes
```
- créer un alias **vm** pour notre machine virtuelle qui utilise par défaut l’utilisateur user et non notre login : 
```
Host vm
    HostName 192.168.194.3
    User user
    ForwardAgent yes
```
- créer un alias **vmjump** pour pouvoir se connecter à otre VM directement depuis notre machine physique (directive ProxyJump). En d’autre terme, la commande ssh vmjump se connectera d’abord à la machine de virtualisation puis, automatiquement à la VM, sans autre manipulation de notre part : 
```
Host vmjump
    HostName 192.168.194.3
    User user
    ForwardAgent yes
    ProxyJump virt
```

Une fois tout cela fait, notre machine est prête à être utilisé et on peut passer à [l'installation d'une base de données](./installation_bdd.md).