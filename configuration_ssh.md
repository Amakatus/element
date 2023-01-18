Afin d'aller plus vite durant la manipulation des différentes machines, on peut créer des scripts, en modifiant le fichier config qui se trouve à l'emplacement : <span style="color:salmon">$HOME/.ssh/config</span>

- créer un alias **virt** pour votre machine de virtualisation. Ainsi, si votre machine de virtualisation est meleze12.iutinfo.fr, alors utiliser la commande ssh virt effectura en réalité l’équivalent de la commande ssh meleze12.iutinfo.fr ;
```
Host virt
    HostName acajou13.iutinfo.fr
    ForwardAgent
```
- effectuer un transfert de votre agent SSH quand vous vous connectez sur la machine de virtualisation. Vous pourez ainsi diffuser votre clé publique sur votre VM et utiliser votre clé privé (via l’agent de votre machine physique) depuis la machine de virtualisation ;
```
Host virt
    HostName acajou13.iutinfo.fr
    ForwardAgent yes
```
- créer un alias **vm** pour votre machine virtuelle qui utilise par défaut l’utilisateur user et non votre login;
```
Host vm
    HostName 192.168.194.3
    User user
    ForwardAgent yes
```
- créer un alias **vmjump** pour pouvoir se connecter à votre VM directement depuis votre machine physique (directive ProxyJump). En d’autre terme, la commande ssh vmjump se connectera d’abord à la machine de virtualisation puis, automatiquement à la VM, sans autre manipulation de votre part.
```
Host vmjump
    HostName 192.168.194.3
    User user
    ForwardAgent yes
    ProxyJump virt
```