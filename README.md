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

Afin de savoir le processus qui utilise le plus le processeur, on utilise la commande `top` qui nous renvoie ceci:

```
PID UTIL.     PR  NI    VIRT    RES    SHR S  %CPU  %MEM    TEMPS+ COM.
559 root      20   0   17996  11068   9252 S   0,3   0,5   0:00.61 sshd
```

Donc le processus qui utilise le plus le processeur est `sshd`.

--------



--------

Pour savoir a quel heure ma machine a démarrée j'utilise la commande `who -b`.
```
root@serveur1:~# who -b
         démarrage système 2024-10-15 22:47
```

L'autre commande permettant de savoir le temps depuis lequel le serveur tourne et la commande `uptime`qui renvoie ceci :

```
root@serveur1:~# uptime
 22:56:53 up 9 min,  2 users,  load average: 0,01, 0,04, 0,02
```

--------

## 3  Arrêt d’un processus
###

## 4 Les tubes

Selon la commande `man`, la différence entre les commandes `tee` et `cat` réside dans leurs fonctions respectives. En effet, `tee` permet de lire des données depuis l'entrée standard et d'écrire ces données à la fois sur la sortie standard et dans des fichiers spécifiés.

En revanche, `cat` (abréviation de "concatenate") est utilisé pour concaténer des fichiers et les afficher sur la sortie standard. Cette commande permet principalement de combiner le contenu de plusieurs fichiers et de l'afficher.

--------

La commande `ls | cat` permet de lister le contenu du répertoire courant.

--------

La commande `ls -l | cat > liste` permet de lister les fichiers et répertoires en détail dans un fichier nommé liste.

--------

La commande `ls -l | tee liste` permet de lister les fichiers et répertoires en détail et envoie le résultat à la fois à l'écran et dans un fichier nommé liste.

--------

La commande `ls -l | tee liste | wc -l` permet de lister les fichiers et répertoires en détail, enregistre cette sortie dans un fichier nommé liste, puis compte le nombre de lignes de cette sortie.

## 5 Journal système rsyslog

Tout d'abord j'ai du installé rsyslog avec la commande `apt install rsyslog`.

Une fois installé 
