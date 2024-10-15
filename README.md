# Compte Rendu TP02

## 1 Secure Shell : SSH
### 1.1 Connection ssh root

```

```

### 1.2 Authentification par clef / Génération de clefs

Pour sécuriser les connexions SSH, j'ai généré une paire de clés à l'aide de la commande `ssh-keygen`. Cette commande a créé deux fichiers :

`id_rsa` : Clé privée, qui doit rester confidentielle.

`id_rsa.pub` : Clé publique, destinée à être partagée avec les serveurs.

Voici ce qu'on obtient une fois la création des clefs:
```
The key's randomart image is:
+---[RSA 3072]----+
|           oo+.=.|
|           ...Oo+|
|            ..+=B|
|             =.EX|
|        S   o +=B|
|         . .   o+|
|          + oo=.*|
|         . o o=X.|
|              oo+|
+----[SHA256]-----+
```

### 1.3 Authentification par clef / Connection serveur

Afin de sécuriser l'accès au serveur et éviter les connexions root par mot de passe, j'ai mis en place l'authentification par clé publique.

```
root@serveur1:/# cd /
root@serveur1:/# cd root/
root@serveur1:~# cd .ssh/
root@serveur1:~/.ssh# touch authorized_keys
root@serveur1:~/.ssh# nano authorized_keys 
root@serveur1:~/.ssh# chmod 700 authorized_keys
```

Pour expliquer l'ensemble des commandes ci-dessus, ont crée un fichier `authorized_keys`, puis on ajoute la clé publique dedans avec la commande `nano` qui permet de modifier le contenue d'un fichier. Et enfin on configure des permissions pour sécuriser le fichier.

### 1.4 Authentification par clef : depuis la machine hote

Pour ce connecter avec la clé on effectue la commande suivante:

`ssh -p 3022 -i .ssh/id_rsa root@127.0.0.1`

Ici on ajoute `-i` qui d'après le manuel correspond au fichier d'identification. (Dans notre cas on utilise une redirection port c'est pour cela qu'il y a le paramètre -p 3022).

### 1.5 Sécuriser

Pour renforcer la sécurité de l'accès SSH, il est nécessaire de désactiver la possibilité de se connecter en tant que root via un mot de passe, en se basant sur les actions effectuées dans le TP 01. Pour ce faire on doit modifier le fichier `sshd_config`.

Donc on execute la commande  `nano /etc/ssh/sshd_config`

Et on change la ligne suivante : 
`PasswordAuthentication no`

Une fois cette commande fini, on execute `systemctl restart sshd` afin de prendre en compte les modifications.

**Petite définition :**
Les attaques de type brute-force SSH consistent en des tentatives répétées et automatiques de se connecter à un serveur SSH en essayant un grand nombre de combinaisons de noms d'utilisateurs et de mots de passe, jusqu'à ce qu'une combinaison valide soit trouvée.

## 2 Processus
### 2.1  Etude des processus UNIX

D'après la commande `man` pour afficher les processus avec les informations demandé on doit executé la commande suivante:

`ps -eo user,pid,%cpu,%mem,stat,start,time,comm`

Ici `-e` sélectionne tous les processus tournant sur la machine et `-o` permet de spécifier les colonnes à afficher.

--------

L'information **TIME** correspond au temps total CPU utilisé par le processus.

--------

## 3 
###

## 4 
###

## 5
