# Compte Rendu TP02

## 1 Secure Shell : SSH
### 1.1 Connection ssh root

Pour pouvoir se connecter en tant que root, il a fallut modifier dans `sshd_config` les lignes suivantes :

```
PasswordAuthentication no
PermitRootLogin prohibit-password
```

par

```
PasswordAuthentication yes
PermitRootLogin yes
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

D'après la commande man, en faisant `ps -p 1`, j'obtient le premier processus lancé après le démarrage du système, soit `systemd`.

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

Afin de savoir le nombre aproximatif de processus créés depuis le démarrage de ma machine, c'est en faisant la commande `top`, je navigue vers le bas et je regarde le PID le plus haut, soit dans mon cas envrions **636**.

## 3  Arrêt d’un processus

Une fois les 2 scripts créés et executer par la commande `./date.sh &` et `./date-toto.sh &` et mis en arrière plan par la combinaison `CTRL Z` il fallait ensuite arreter les 2 processus.

Pour ce faire il faut ramener au premier plan les processus. Ainsi on execute les commandes suivante:

`job` : afin de voir les processus en arrière plan.

`fg %1` et `fg %2` Pour ramener au premier plan le processus.

Enfin une fois le processus au premier plan il suffit de faire la combinaison `CTRL C`.

--------

En utilisant les commandes `ps` et `kill`, cela revient au même:

`ps` : afin de voir les processus en arrière plan et leur PID.

`kill -9 <PID>` Pour quitter le processus en fonction du PID.

--------

**date.sh**
Le script date.sh est conçu pour afficher l'heure actuelle en continu. Il commence par indiquer qu'il doit être exécuté dans un environnement shell. Ensuite, une boucle infinie est utilisée pour répéter les instructions. À chaque itération, le script se met en pause pendant 1 seconde. Ensuite, il affiche le mot "date" suivi de l'heure actuelle formatée en heures, minutes et secondes.


**date-toto.sh**
Le script date-toto.sh fonctionne de manière similaire au premier script, mais il affiche une heure qui correspond à il y a 5 heures. Comme dans le premier script, il commence par #!/bin/sh pour spécifier le shell. La boucle infinie permet d'exécuter les instructions en continu, avec une pause d'une seconde entre chaque itération.

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

Une fois installé on execute la commade `systemctl status rsyslog` afin de savoir le PID du deamon et d'autres informations falcultatives.

```
root@serveur1:~# systemctl status rsyslog
● rsyslog.service - System Logging Service
     Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; preset: enab>
     Active: active (running) since Tue 2024-10-15 23:17:17 CEST; 26s ago
TriggeredBy: ● syslog.socket
       Docs: man:rsyslogd(8)
             man:rsyslog.conf(5)
             https://www.rsyslog.com/doc/
   Main PID: 919 (rsyslogd)
      Tasks: 4 (limit: 2315)
     Memory: 1.2M
        CPU: 9ms
     CGroup: /system.slice/rsyslog.service
             └─919 /usr/sbin/rsyslogd -n -iNONE

oct. 15 23:17:17 serveur1 systemd[1]: Starting rsyslog.service - System Logging>
oct. 15 23:17:17 serveur1 systemd[1]: Started rsyslog.service - System Logging >
oct. 15 23:17:17 serveur1 rsyslogd[919]: imuxsock: Acquired UNIX socket '/run/s>
oct. 15 23:17:17 serveur1 rsyslogd[919]: [origin software="rsyslogd" swVersion=>
lines 1-18/18 (END)
```

Ainsi le PID du deamon est **919**.

--------

En regardant la config de `/etc/rsyslog.conf`, **rsyslog** écrit les messages issus des services standards dans `/var/log/syslog`.
La plupart des autres messages sont écrit dans les répertoires ci-dessous.

```
#Log commonly used facilities to their own log file
auth,authpriv.*                 /var/log/auth.log
cron.*                          -/var/log/cron.log
kern.*                          -/var/log/kern.log
mail.*                          -/var/log/mail.log
user.*                          -/var/log/user.log
```

Pour vérifier le contenu de ces fichiers on peut faire les commandes suivantes :

```
cat /var/log/syslog
cat /var/log/cron
cat /var/log/kern
cat /var/log/mail
cat /var/log/user
```

--------

`Cron` est un programme qui permet aux utilisateurs des systèmes Unix d’exécuter automatiquement des scripts, des commandes ou des logiciels à une date et une heure spécifiée à l’avance, ou selon un cycle défini à l’avance.

--------

D'après la commande `man`, `tail -f` permet d'afficher la dernière partie de fichiers, ainsi depuis un autre terminal, j'ai pu observé des **logs de redémarrage**.

--------

Le fichier `/etc/logrotate.conf` est utilisé pour configurer la rotation des fichiers journaux. La rotation des journaux est une méthode de gestion des fichiers de log qui permet de limiter leur taille et de conserver l'historique des journaux.

--------

La commande `dmesg` renvoie ceci:

```
root@serveur1:~# dmesg
[    0.000000] Linux version 6.1.0-25-amd64 (debian-kernel@lists.debian.org) (gcc-12 (Debian 12.2.0-14) 12.2.0, GNU ld (GNU Binutils for Debian) 2.40) #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26)
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-6.1.0-25-amd64 root=UUID=65533216-0649-4c98-9526-28f9f962f937 ro quiet
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
[    0.000000] BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x000000007ffeffff] usable
[    0.000000] BIOS-e820: [mem 0x000000007fff0000-0x000000007fffffff] ACPI data
[    0.000000] BIOS-e820: [mem 0x00000000fec00000-0x00000000fec00fff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000fee00000-0x00000000fee00fff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
[    0.000000] NX (Execute Disable) protection: active
[    0.000000] SMBIOS 2.5 present.
[    0.000000] DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
[    0.000000] Hypervisor detected: KVM
[    0.000000] kvm-clock: Using msrs 4b564d01 and 4b564d00
[    0.000004] kvm-clock: using sched offset of 12206325609 cycles
[    0.000007] clocksource: kvm-clock: mask: 0xffffffffffffffff max_cycles: 0x1cd42e4dffb, max_idle_ns: 881590591483 ns
[    0.000011] tsc: Detected 1398.457 MHz processor
[    0.001956] e820: update [mem 0x00000000-0x00000fff] usable ==> reserved
[    0.001960] e820: remove [mem 0x000a0000-0x000fffff] usable
[    0.001966] last_pfn = 0x7fff0 max_arch_pfn = 0x400000000
[    0.002915] x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT  
[    0.023627] found SMP MP-table at [mem 0x0009fbf0-0x0009fbff]
[    0.024119] RAMDISK: [mem 0x34501000-0x36277fff]
[    0.024123] ACPI: Early table checksum verification disabled
[    0.024128] ACPI: RSDP 0x00000000000E0000 000024 (v02 VBOX  )
[    0.024134] ACPI: XSDT 0x000000007FFF0030 00003C (v01 VBOX   VBOXXSDT 00000001 ASL  00000061)
[    0.024141] ACPI: FACP 0x000000007FFF00F0 0000F4 (v04 VBOX   VBOXFACP 00000001 ASL  00000061)
[    0.024148] ACPI: DSDT 0x000000007FFF0610 002353 (v02 VBOX   VBOXBIOS 00000002 INTL 20240322)
[    0.024153] ACPI: FACS 0x000000007FFF0200 000040
[    0.024156] ACPI: FACS 0x000000007FFF0200 000040
[    0.024160] ACPI: APIC 0x000000007FFF0240 000054 (v02 VBOX   VBOXAPIC 00000001 ASL  00000061)
[    0.024165] ACPI: SSDT 0x000000007FFF02A0 00036C (v01 VBOX   VBOXCPUT 00000002 INTL 20240322)
[    0.024168] ACPI: Reserving FACP table memory at [mem 0x7fff00f0-0x7fff01e3]
[    0.024170] ACPI: Reserving DSDT table memory at [mem 0x7fff0610-0x7fff2962]
[    0.024171] ACPI: Reserving FACS table memory at [mem 0x7fff0200-0x7fff023f]
[    0.024172] ACPI: Reserving FACS table memory at [mem 0x7fff0200-0x7fff023f]
[    0.024173] ACPI: Reserving APIC table memory at [mem 0x7fff0240-0x7fff0293]
[    0.024174] ACPI: Reserving SSDT table memory at [mem 0x7fff02a0-0x7fff060b]
[    0.025871] No NUMA configuration found
[    0.025873] Faking a node at [mem 0x0000000000000000-0x000000007ffeffff]
[    0.025882] NODE_DATA(0) allocated [mem 0x7ffc5000-0x7ffeffff]
[    0.026099] Zone ranges:
[    0.026101]   DMA      [mem 0x0000000000001000-0x0000000000ffffff]
[    0.026103]   DMA32    [mem 0x0000000001000000-0x000000007ffeffff]
[    0.026105]   Normal   empty
[    0.026106]   Device   empty
[    0.026107] Movable zone start for each node
[    0.026109] Early memory node ranges
[    0.026109]   node   0: [mem 0x0000000000001000-0x000000000009efff]
[    0.026111]   node   0: [mem 0x0000000000100000-0x000000007ffeffff]
[    0.026113] Initmem setup node 0 [mem 0x0000000000001000-0x000000007ffeffff]
[    0.026119] On node 0, zone DMA: 1 pages in unavailable ranges
[    0.026150] On node 0, zone DMA: 97 pages in unavailable ranges
[    0.026693] On node 0, zone DMA32: 16 pages in unavailable ranges
[    0.029053] ACPI: PM-Timer IO Port: 0x4008
[    0.029316] IOAPIC[0]: apic_id 1, version 32, address 0xfec00000, GSI 0-23
[    0.029322] ACPI: INT_SRC_OVR (bus 0 bus_irq 0 global_irq 2 dfl dfl)
[    0.029325] ACPI: INT_SRC_OVR (bus 0 bus_irq 9 global_irq 9 low level)
[    0.029331] ACPI: Using ACPI (MADT) for SMP configuration information
[    0.029356] smpboot: Allowing 1 CPUs, 0 hotplug CPUs
[    0.029474] PM: hibernation: Registered nosave memory: [mem 0x00000000-0x00000fff]
[    0.029477] PM: hibernation: Registered nosave memory: [mem 0x0009f000-0x0009ffff]
[    0.029478] PM: hibernation: Registered nosave memory: [mem 0x000a0000-0x000effff]
[    0.029479] PM: hibernation: Registered nosave memory: [mem 0x000f0000-0x000fffff]
[    0.029481] [mem 0x80000000-0xfebfffff] available for PCI devices
[    0.029482] Booting paravirtualized kernel on KVM
[    0.029485] clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
[    0.036150] setup_percpu: NR_CPUS:8192 nr_cpumask_bits:1 nr_cpu_ids:1 nr_node_ids:1
[    0.039486] percpu: Embedded 61 pages/cpu s212992 r8192 d28672 u2097152
[    0.039498] pcpu-alloc: s212992 r8192 d28672 u2097152 alloc=1*2097152
[    0.039501] pcpu-alloc: [0] 0 
[    0.039603] kvm-guest: PV spinlocks disabled, single CPU
[    0.039613] Fallback order for Node 0: 0 
[    0.039616] Built 1 zonelists, mobility grouping on.  Total pages: 515824
[    0.039618] Policy zone: DMA32
[    0.039620] Kernel command line: BOOT_IMAGE=/boot/vmlinuz-6.1.0-25-amd64 root=UUID=65533216-0649-4c98-9526-28f9f962f937 ro quiet
[    0.039699] Unknown kernel command line parameters "BOOT_IMAGE=/boot/vmlinuz-6.1.0-25-amd64", will be passed to user space.
[    0.039733] random: crng init done
[    0.043658] Dentry cache hash table entries: 262144 (order: 9, 2097152 bytes, linear)
[    0.043777] Inode-cache hash table entries: 131072 (order: 8, 1048576 bytes, linear)
[    0.044010] mem auto-init: stack:all(zero), heap alloc:on, heap free:off
[    0.046461] Memory: 260860K/2096696K available (14342K kernel code, 2335K rwdata, 9072K rodata, 2796K init, 17396K bss, 120444K reserved, 0K cma-reserved)
[    0.046704] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.046726] Kernel/User page tables isolation: enabled
[    0.046758] ftrace: allocating 40246 entries in 158 pages
[    0.054103] ftrace: allocated 158 pages with 5 groups
[    0.054872] Dynamic Preempt: voluntary
[    0.054899] rcu: Preemptible hierarchical RCU implementation.
[    0.054900] rcu: 	RCU restricting CPUs from NR_CPUS=8192 to nr_cpu_ids=1.
[    0.054902] 	Trampoline variant of Tasks RCU enabled.
[    0.054903] 	Rude variant of Tasks RCU enabled.
[    0.054903] 	Tracing variant of Tasks RCU enabled.
[    0.054904] rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
[    0.054905] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=1
[    0.059701] NR_IRQS: 524544, nr_irqs: 256, preallocated irqs: 16
[    0.060617] rcu: srcu_init: Setting srcu_struct sizes based on contention.
[    0.105257] Console: colour VGA+ 80x25
[    0.105273] printk: console [tty0] enabled
[    0.105301] ACPI: Core revision 20220331
[    0.105563] APIC: Switch to symmetric I/O mode setup
[    0.107556] x2apic enabled
[    0.109500] Switched APIC routing to physical x2apic.
[    0.118949] ..TIMER: vector=0x30 apic1=0 pin1=2 apic2=-1 pin2=-1
[    0.119300] clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x14286eb5467, max_idle_ns: 440795222954 ns
[    0.119311] Calibrating delay loop (skipped) preset value.. 2796.91 BogoMIPS (lpj=5593828)
[    0.121069] process: using mwait in idle threads
[    0.121106] Last level iTLB entries: 4KB 64, 2MB 8, 4MB 8
[    0.121108] Last level dTLB entries: 4KB 64, 2MB 0, 4MB 0, 1GB 4
[    0.121146] Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
[    0.121150] Spectre V2 : Mitigation: Retpolines
[    0.121150] Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
[    0.121151] Spectre V2 : Spectre v2 / SpectreRSB : Filling RSB on VMEXIT
[    0.121152] RETBleed: WARNING: Spectre v2 mitigation leaves CPU vulnerable to RETBleed attacks, data leaks possible!
[    0.123313] RETBleed: Vulnerable
[    0.123315] Speculative Store Bypass: Vulnerable
[    0.123321] MDS: Mitigation: Clear CPU buffers
[    0.123322] MMIO Stale Data: Mitigation: Clear CPU buffers
[    0.123323] SRBDS: Unknown: Dependent on hypervisor status
[    0.123324] GDS: Unknown: Dependent on hypervisor status
[    0.123439] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
[    0.123442] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
[    0.123443] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
[    0.123444] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
[    0.123446] x86/fpu: Enabled xstate features 0x7, context size is 832 bytes, using 'standard' format.
[    0.134235] Freeing SMP alternatives memory: 36K
[    0.134242] pid_max: default: 32768 minimum: 301
[    0.134277] LSM: Security Framework initializing
[    0.134290] landlock: Up and running.
[    0.134291] Yama: disabled by default; enable with sysctl kernel.yama.*
[    0.134315] AppArmor: AppArmor initialized
[    0.134317] TOMOYO Linux initialized
[    0.134322] LSM support for eBPF active
[    0.134344] Mount-cache hash table entries: 4096 (order: 3, 32768 bytes, linear)
[    0.134348] Mountpoint-cache hash table entries: 4096 (order: 3, 32768 bytes, linear)
[    0.251309] APIC calibration not consistent with PM-Timer: 112ms instead of 100ms
[    0.251315] APIC delta adjusted to PM-Timer: 6421184 (7250413)
[    0.251817] smpboot: CPU0: Intel(R) Core(TM) i5-8257U CPU @ 1.40GHz (family: 0x6, model: 0x8e, stepping: 0xa)
[    0.251976] cblist_init_generic: Setting adjustable number of callback queues.
[    0.251978] cblist_init_generic: Setting shift to 0 and lim to 1.
[    0.251993] cblist_init_generic: Setting adjustable number of callback queues.
[    0.251994] cblist_init_generic: Setting shift to 0 and lim to 1.
[    0.252007] cblist_init_generic: Setting adjustable number of callback queues.
[    0.252008] cblist_init_generic: Setting shift to 0 and lim to 1.
[    0.252019] Performance Events: unsupported p6 CPU model 142 no PMU driver, software events only.
[    0.252056] signal: max sigframe size: 1776
[    0.252084] rcu: Hierarchical SRCU implementation.
[    0.252085] rcu: 	Max phase no-delay instances is 1000.
[    0.252343] NMI watchdog: Perf NMI watchdog permanently disabled
[    0.252383] smp: Bringing up secondary CPUs ...
[    0.252384] smp: Brought up 1 node, 1 CPU
[    0.252386] smpboot: Max logical packages: 1
[    0.252387] smpboot: Total of 1 processors activated (2796.91 BogoMIPS)
[    0.256248] node 0 deferred pages initialised in 4ms
[    0.256443] devtmpfs: initialized
[    0.256492] x86/mm: Memory block size: 128MB
[    0.256883] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.256889] futex hash table entries: 256 (order: 2, 16384 bytes, linear)
[    0.256923] pinctrl core: initialized pinctrl subsystem
[    0.258377] NET: Registered PF_NETLINK/PF_ROUTE protocol family
[    0.258470] DMA: preallocated 256 KiB GFP_KERNEL pool for atomic allocations
[    0.258489] DMA: preallocated 256 KiB GFP_KERNEL|GFP_DMA pool for atomic allocations
[    0.258506] DMA: preallocated 256 KiB GFP_KERNEL|GFP_DMA32 pool for atomic allocations
[    0.258516] audit: initializing netlink subsys (disabled)
[    0.258644] thermal_sys: Registered thermal governor 'fair_share'
[    0.258646] thermal_sys: Registered thermal governor 'bang_bang'
[    0.258647] thermal_sys: Registered thermal governor 'step_wise'
[    0.258647] thermal_sys: Registered thermal governor 'user_space'
[    0.258648] thermal_sys: Registered thermal governor 'power_allocator'
[    0.258659] cpuidle: using governor ladder
[    0.258662] cpuidle: using governor menu
[    0.258829] acpiphp: ACPI Hot Plug PCI Controller Driver version: 0.5
[    0.259306] PCI: Using configuration type 1 for base access
[    0.259306] kprobes: kprobe jump-optimization is enabled. All kprobes are optimized if possible.
[    0.259306] audit: type=2000 audit(1729026990.165:1): state=initialized audit_enabled=0 res=1
[    0.271306] HugeTLB: registered 2.00 MiB page size, pre-allocated 0 pages
[    0.271306] HugeTLB: 28 KiB vmemmap can be freed for a 2.00 MiB page
[    0.271306] ACPI: Added _OSI(Module Device)
[    0.271306] ACPI: Added _OSI(Processor Device)
[    0.271306] ACPI: Added _OSI(3.0 _SCP Extensions)
[    0.271306] ACPI: Added _OSI(Processor Aggregator Device)
[    0.271306] ACPI: 2 ACPI AML tables successfully acquired and loaded
[    0.273054] ACPI: Interpreter enabled
[    0.273064] ACPI: PM: (supports S0 S5)
[    0.273066] ACPI: Using IOAPIC for interrupt routing
[    0.273425] PCI: Using host bridge windows from ACPI; if necessary, use "pci=nocrs" and report a bug
[    0.273427] PCI: Using E820 reservations for host bridge windows
[    0.273558] ACPI: Enabled 2 GPEs in block 00 to 07
[    0.276085] ACPI: PCI Root Bridge [PCI0] (domain 0000 [bus 00-ff])
[    0.276093] acpi PNP0A03:00: _OSC: OS supports [ASPM ClockPM Segments MSI HPX-Type3]
[    0.276096] acpi PNP0A03:00: _OSC: not requesting OS control; OS requires [ExtendedConfig ASPM ClockPM MSI]
[    0.276769] acpi PNP0A03:00: fail to add MMCONFIG information, can't access extended PCI configuration space under this bridge.
[    0.277504] PCI host bridge to bus 0000:00
[    0.277507] pci_bus 0000:00: root bus resource [io  0x0000-0x0cf7 window]
[    0.277510] pci_bus 0000:00: root bus resource [io  0x0d00-0xffff window]
[    0.277512] pci_bus 0000:00: root bus resource [mem 0x000a0000-0x000bffff window]
[    0.277514] pci_bus 0000:00: root bus resource [mem 0x80000000-0xfdffffff window]
[    0.277516] pci_bus 0000:00: root bus resource [bus 00-ff]
[    0.277962] pci 0000:00:00.0: [8086:1237] type 00 class 0x060000
[    0.279306] pci 0000:00:01.0: [8086:7000] type 00 class 0x060100
[    0.279306] pci 0000:00:01.1: [8086:7111] type 00 class 0x01018a
[    0.279306] pci 0000:00:01.1: reg 0x20: [io  0xd000-0xd00f]
[    0.279306] pci 0000:00:01.1: legacy IDE quirk: reg 0x10: [io  0x01f0-0x01f7]
[    0.279306] pci 0000:00:01.1: legacy IDE quirk: reg 0x14: [io  0x03f6]
[    0.279306] pci 0000:00:01.1: legacy IDE quirk: reg 0x18: [io  0x0170-0x0177]
[    0.279306] pci 0000:00:01.1: legacy IDE quirk: reg 0x1c: [io  0x0376]
[    0.279306] pci 0000:00:02.0: [15ad:0405] type 00 class 0x030000
[    0.283311] pci 0000:00:02.0: reg 0x10: [io  0xd010-0xd01f]
[    0.287313] pci 0000:00:02.0: reg 0x14: [mem 0xe0000000-0xe0ffffff pref]
[    0.291306] pci 0000:00:02.0: reg 0x18: [mem 0xf0000000-0xf01fffff]
[    0.307306] pci 0000:00:02.0: Video device with shadowed ROM at [mem 0x000c0000-0x000dffff]
[    0.307306] pci 0000:00:03.0: [8086:100e] type 00 class 0x020000
[    0.307306] pci 0000:00:03.0: reg 0x10: [mem 0xf0200000-0xf021ffff]
[    0.307306] pci 0000:00:03.0: reg 0x18: [io  0xd020-0xd027]
[    0.307306] pci 0000:00:04.0: [80ee:cafe] type 00 class 0x088000
[    0.307306] pci 0000:00:04.0: reg 0x10: [io  0xd040-0xd05f]
[    0.307306] pci 0000:00:04.0: reg 0x14: [mem 0xf0400000-0xf07fffff]
[    0.307306] pci 0000:00:04.0: reg 0x18: [mem 0xf0800000-0xf0803fff pref]
[    0.310710] pci 0000:00:05.0: [8086:2415] type 00 class 0x040100
[    0.311180] pci 0000:00:05.0: reg 0x10: [io  0xd100-0xd1ff]
[    0.311306] pci 0000:00:05.0: reg 0x14: [io  0xd200-0xd23f]
[    0.311306] pci 0000:00:06.0: [106b:003f] type 00 class 0x0c0310
[    0.311306] pci 0000:00:06.0: reg 0x10: [mem 0xf0804000-0xf0804fff]
[    0.315264] pci 0000:00:07.0: [8086:7113] type 00 class 0x068000
[    0.315306] pci 0000:00:07.0: quirk: [io  0x4000-0x403f] claimed by PIIX4 ACPI
[    0.315306] pci 0000:00:07.0: quirk: [io  0x4100-0x410f] claimed by PIIX4 SMB
[    0.315306] pci 0000:00:0b.0: [8086:265c] type 00 class 0x0c0320
[    0.315306] pci 0000:00:0b.0: reg 0x10: [mem 0xf0805000-0xf0805fff]
[    0.318873] pci 0000:00:0d.0: [8086:2829] type 00 class 0x010601
[    0.319306] pci 0000:00:0d.0: reg 0x10: [io  0xd240-0xd247]
[    0.319306] pci 0000:00:0d.0: reg 0x14: [io  0xd248-0xd24b]
[    0.319306] pci 0000:00:0d.0: reg 0x18: [io  0xd250-0xd257]
[    0.319306] pci 0000:00:0d.0: reg 0x1c: [io  0xd258-0xd25b]
[    0.319306] pci 0000:00:0d.0: reg 0x20: [io  0xd260-0xd26f]
[    0.319306] pci 0000:00:0d.0: reg 0x24: [mem 0xf0806000-0xf0807fff]
[    0.323306] ACPI: PCI: Interrupt link LNKA configured for IRQ 11
[    0.323306] ACPI: PCI: Interrupt link LNKB configured for IRQ 10
[    0.323306] ACPI: PCI: Interrupt link LNKC configured for IRQ 9
[    0.323306] ACPI: PCI: Interrupt link LNKD configured for IRQ 11
[    0.323306] iommu: Default domain type: Translated 
[    0.323306] iommu: DMA domain TLB invalidation policy: lazy mode 
[    0.323306] pps_core: LinuxPPS API ver. 1 registered
[    0.323306] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.323306] PTP clock support registered
[    0.323306] EDAC MC: Ver: 3.0.0
[    0.323306] NetLabel: Initializing
[    0.323306] NetLabel:  domain hash size = 128
[    0.323306] NetLabel:  protocols = UNLABELED CIPSOv4 CALIPSO
[    0.323306] NetLabel:  unlabeled traffic allowed by default
[    0.323306] PCI: Using ACPI for IRQ routing
[    0.323306] PCI: pci_cache_line_size set to 64 bytes
[    0.323306] e820: reserve RAM buffer [mem 0x0009fc00-0x0009ffff]
[    0.323306] e820: reserve RAM buffer [mem 0x7fff0000-0x7fffffff]
[    0.323306] pci 0000:00:02.0: vgaarb: setting as boot VGA device
[    0.323306] pci 0000:00:02.0: vgaarb: bridge control possible
[    0.323306] pci 0000:00:02.0: vgaarb: VGA device added: decodes=io+mem,owns=io+mem,locks=none
[    0.323306] vgaarb: loaded
[    0.323306] clocksource: Switched to clocksource kvm-clock
[    0.323306] VFS: Disk quotas dquot_6.6.0
[    0.323306] VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
[    0.323306] AppArmor: AppArmor Filesystem Enabled
[    0.323306] pnp: PnP ACPI init
[    0.323306] pnp: PnP ACPI: found 2 devices
[    0.340641] clocksource: acpi_pm: mask: 0xffffff max_cycles: 0xffffff, max_idle_ns: 2085701024 ns
[    0.340727] NET: Registered PF_INET protocol family
[    0.340809] IP idents hash table entries: 32768 (order: 6, 262144 bytes, linear)
[    0.342645] tcp_listen_portaddr_hash hash table entries: 1024 (order: 2, 16384 bytes, linear)
[    0.342668] Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.342680] TCP established hash table entries: 16384 (order: 5, 131072 bytes, linear)
[    0.342730] TCP bind hash table entries: 16384 (order: 7, 524288 bytes, linear)
[    0.342776] TCP: Hash tables configured (established 16384 bind 16384)
[    0.342827] MPTCP token hash table entries: 2048 (order: 3, 49152 bytes, linear)
[    0.342917] UDP hash table entries: 1024 (order: 3, 32768 bytes, linear)
[    0.342987] UDP-Lite hash table entries: 1024 (order: 3, 32768 bytes, linear)
[    0.343027] NET: Registered PF_UNIX/PF_LOCAL protocol family
[    0.343037] NET: Registered PF_XDP protocol family
[    0.343050] pci_bus 0000:00: resource 4 [io  0x0000-0x0cf7 window]
[    0.343054] pci_bus 0000:00: resource 5 [io  0x0d00-0xffff window]
[    0.343056] pci_bus 0000:00: resource 6 [mem 0x000a0000-0x000bffff window]
[    0.343058] pci_bus 0000:00: resource 7 [mem 0x80000000-0xfdffffff window]
[    0.343126] pci 0000:00:00.0: Limiting direct PCI/PCI transfers
[    0.343277] PCI: CLS 0 bytes, default 64
[    0.343277] Trying to unpack rootfs image as initramfs...
[    0.351879] clocksource: tsc: mask: 0xffffffffffffffff max_cycles: 0x14286eb5467, max_idle_ns: 440795222954 ns
[    0.351894] clocksource: Switched to clocksource tsc
[    0.351927] platform rtc_cmos: registered platform RTC device (no PNP device found)
[    0.352196] Initialise system trusted keyrings
[    0.352206] Key type blacklist registered
[    0.352258] workingset: timestamp_bits=36 max_order=19 bucket_order=0
[    0.353513] zbud: loaded
[    0.353843] integrity: Platform Keyring initialized
[    0.353848] integrity: Machine keyring initialized
[    0.353850] Key type asymmetric registered
[    0.353852] Asymmetric key parser 'x509' registered
[    0.893900] Freeing initrd memory: 30172K
[    0.903964] alg: self-tests for CTR-KDF (hmac(sha256)) passed
[    0.903964] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 248)
[    0.909349] io scheduler mq-deadline registered
[    0.910405] shpchp: Standard Hot Plug PCI Controller Driver version: 0.4
[    0.911169] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    0.911898] Linux agpgart interface v0.103
[    0.912012] AMD-Vi: AMD IOMMUv2 functionality not available on this system - This is not a bug.
[    0.912212] i8042: PNP: PS/2 Controller [PNP0303:PS2K,PNP0f03:PS2M] at 0x60,0x64 irq 1,12
[    0.914822] serio: i8042 KBD port at 0x60,0x64 irq 1
[    0.914832] serio: i8042 AUX port at 0x60,0x64 irq 12
[    0.914919] mousedev: PS/2 mouse device common for all mice
[    0.914919] input: AT Translated Set 2 keyboard as /devices/platform/i8042/serio0/input/input0
[    0.919948] rtc_cmos rtc_cmos: registered as rtc0
[    0.920320] rtc_cmos rtc_cmos: setting system clock to 2024-10-15T21:16:17 UTC (1729026977)
[    0.920482] rtc_cmos rtc_cmos: alarms up to one day, 114 bytes nvram
[    0.920497] intel_pstate: CPU model not supported
[    0.920528] ledtrig-cpu: registered to indicate activity on CPUs
[    0.933694] NET: Registered PF_INET6 protocol family
[    0.943718] Segment Routing with IPv6
[    0.943718] In-situ OAM (IOAM) with IPv6
[    0.943718] mip6: Mobile IPv6
[    0.943718] NET: Registered PF_PACKET protocol family
[    0.943718] mpls_gso: MPLS GSO support
[    0.943718] IPI shorthand broadcast: enabled
[    0.943718] sched_clock: Marking stable (885160489, 54557564)->(1059558991, -119840938)
[    0.948545] registered taskstats version 1
[    0.948555] Loading compiled-in X.509 certificates
[    0.969479] Loaded X.509 cert 'Debian Secure Boot CA: 6ccece7e4c6c0d1f6149f3dd27dfcc5cbb419ea1'
[    0.969817] Loaded X.509 cert 'Debian Secure Boot Signer 2022 - linux: 14011249c2675ea8e5148542202005810584b25f'
[    0.969959] zswap: loaded using pool lzo/zbud
[    0.970066] Key type .fscrypt registered
[    0.970067] Key type fscrypt-provisioning registered
[    0.977807] Key type encrypted registered
[    0.977813] AppArmor: AppArmor sha1 policy hashing enabled
[    0.977824] ima: No TPM chip found, activating TPM-bypass!
[    0.977826] ima: Allocated hash algorithm: sha256
[    0.977836] ima: No architecture policies found
[    0.977847] evm: Initialising EVM extended attributes:
[    0.977848] evm: security.selinux
[    0.977849] evm: security.SMACK64 (disabled)
[    0.977850] evm: security.SMACK64EXEC (disabled)
[    0.977850] evm: security.SMACK64TRANSMUTE (disabled)
[    0.977851] evm: security.SMACK64MMAP (disabled)
[    0.977852] evm: security.apparmor
[    0.977853] evm: security.ima
[    0.977853] evm: security.capability
[    0.977854] evm: HMAC attrs: 0x1
[    1.085489] clk: Disabling unused clocks
[    1.086739] Freeing unused decrypted memory: 2036K
[    1.087105] Freeing unused kernel image (initmem) memory: 2796K
[    1.087138] Write protecting the kernel read-only data: 26624k
[    1.087521] Freeing unused kernel image (text/rodata gap) memory: 2040K
[    1.087686] Freeing unused kernel image (rodata/data gap) memory: 1168K
[    1.152156] x86/mm: Checked W+X mappings: passed, no W+X pages found.
[    1.152162] x86/mm: Checking user space page tables
[    1.214701] x86/mm: Checked W+X mappings: passed, no W+X pages found.
[    1.214732] Run /init as init process
[    1.214735]   with arguments:
[    1.214736]     /init
[    1.214737]   with environment:
[    1.214738]     HOME=/
[    1.214739]     TERM=linux
[    1.214740]     BOOT_IMAGE=/boot/vmlinuz-6.1.0-25-amd64
[    1.363062] ACPI: battery: Slot [BAT0] (battery present)
[    1.366492] ACPI: video: Video Device [GFX0] (multi-head: yes  rom: no  post: no)
[    1.384568] e1000: Intel(R) PRO/1000 Network Driver
[    1.384572] e1000: Copyright (c) 1999-2006 Intel Corporation.
[    1.399279] piix4_smbus 0000:00:07.0: SMBus Host Controller at 0x4100, revision 0
[    1.432851] input: Video Bus as /devices/LNXSYSTM:00/LNXSYBUS:00/PNP0A03:00/LNXVIDEO:00/input/input2
[    1.472371] SCSI subsystem initialized
[    1.513495] ACPI: bus type USB registered
[    1.518615] usbcore: registered new interface driver usbfs
[    1.518739] usbcore: registered new interface driver hub
[    1.518757] usbcore: registered new device driver usb
[    1.548217] ehci-pci 0000:00:0b.0: EHCI Host Controller
[    1.548227] ehci-pci 0000:00:0b.0: new USB bus registered, assigned bus number 1
[    1.549187] ehci-pci 0000:00:0b.0: irq 19, io mem 0xf0805000
[    1.550440] libata version 3.00 loaded.
[    1.555114] ahci 0000:00:0d.0: version 3.0
[    1.557856] ata_piix 0000:00:01.1: version 2.13
[    1.559493] ahci 0000:00:0d.0: SSS flag set, parallel bus scan disabled
[    1.560274] ahci 0000:00:0d.0: AHCI 0001.0100 32 slots 2 ports 3 Gbps 0x3 impl SATA mode
[    1.560278] ahci 0000:00:0d.0: flags: 64bit ncq stag only ccc 
[    1.567831] ehci-pci 0000:00:0b.0: USB 2.0 started, EHCI 1.00
[    1.567940] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002, bcdDevice= 6.01
[    1.567943] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    1.567945] usb usb1: Product: EHCI Host Controller
[    1.567947] usb usb1: Manufacturer: Linux 6.1.0-25-amd64 ehci_hcd
[    1.567948] usb usb1: SerialNumber: 0000:00:0b.0
[    1.568100] hub 1-0:1.0: USB hub found
[    1.568139] hub 1-0:1.0: 12 ports detected
[    1.570446] ohci-pci 0000:00:06.0: OHCI PCI host controller
[    1.570454] ohci-pci 0000:00:06.0: new USB bus registered, assigned bus number 2
[    1.570770] ohci-pci 0000:00:06.0: irq 22, io mem 0xf0804000
[    1.573896] scsi host0: ata_piix
[    1.574375] scsi host2: ahci
[    1.574687] scsi host1: ata_piix
[    1.574718] ata1: PATA max UDMA/33 cmd 0x1f0 ctl 0x3f6 bmdma 0xd000 irq 14
[    1.574720] ata2: PATA max UDMA/33 cmd 0x170 ctl 0x376 bmdma 0xd008 irq 15
[    1.590188] scsi host3: ahci
[    1.590315] ata3: SATA max UDMA/133 abar m8192@0xf0806000 port 0xf0806100 irq 21
[    1.590339] ata4: SATA max UDMA/133 abar m8192@0xf0806000 port 0xf0806180 irq 21
[    1.636523] usb usb2: New USB device found, idVendor=1d6b, idProduct=0001, bcdDevice= 6.01
[    1.636532] usb usb2: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    1.636534] usb usb2: Product: OHCI PCI host controller
[    1.636536] usb usb2: Manufacturer: Linux 6.1.0-25-amd64 ohci_hcd
[    1.636538] usb usb2: SerialNumber: 0000:00:06.0
[    1.637194] hub 2-0:1.0: USB hub found
[    1.637284] hub 2-0:1.0: 12 ports detected
[    1.799216] input: ImExPS/2 Generic Explorer Mouse as /devices/platform/i8042/serio1/input/input3
[    1.907750] ata3: SATA link up 3.0 Gbps (SStatus 123 SControl 300)
[    1.908262] ata3.00: ATA-6: VBOX HARDDISK, 1.0, max UDMA/133
[    1.908266] ata3.00: 41943040 sectors, multi 128: LBA48 NCQ (depth 32)
[    1.908676] ata3.00: configured for UDMA/133
[    1.908799] scsi 2:0:0:0: Direct-Access     ATA      VBOX HARDDISK    1.0  PQ: 0 ANSI: 5
[    2.092165] usb 2-1: new full-speed USB device number 2 using ohci-pci
[    2.133167] e1000 0000:00:03.0 eth0: (PCI:33MHz:32-bit) 08:00:27:12:3e:86
[    2.133176] e1000 0000:00:03.0 eth0: Intel(R) PRO/1000 Network Connection
[    2.231468] ata4: SATA link up 3.0 Gbps (SStatus 123 SControl 300)
[    2.232547] ata4.00: ATAPI: VBOX CD-ROM, 1.0, max UDMA/133
[    2.232555] ata4.00: applying bridge limits
[    2.233885] ata4.00: configured for UDMA/100
[    2.235621] scsi 3:0:0:0: CD-ROM            VBOX     CD-ROM           1.0  PQ: 0 ANSI: 5
[    2.263763] e1000 0000:00:03.0 enp0s3: renamed from eth0
[    2.269412] sd 2:0:0:0: [sda] 41943040 512-byte logical blocks: (21.5 GB/20.0 GiB)
[    2.269419] sd 2:0:0:0: [sda] Write Protect is off
[    2.269421] sd 2:0:0:0: [sda] Mode Sense: 00 3a 00 00
[    2.269425] sd 2:0:0:0: [sda] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
[    2.269433] sd 2:0:0:0: [sda] Preferred minimum I/O size 512 bytes
[    2.276666]  sda: sda1 sda2 sda3 sda4
[    2.276823] sd 2:0:0:0: [sda] Attached SCSI disk
[    2.320714] sr 3:0:0:0: [sr0] scsi-1 drive
[    2.320721] cdrom: Uniform CD-ROM driver Revision: 3.20
[    2.356799] sr 3:0:0:0: Attached scsi CD-ROM sr0
[    2.423424] usb 2-1: New USB device found, idVendor=80ee, idProduct=0021, bcdDevice= 1.00
[    2.423430] usb 2-1: New USB device strings: Mfr=1, Product=3, SerialNumber=0
[    2.423432] usb 2-1: Product: USB Tablet
[    2.423434] usb 2-1: Manufacturer: VirtualBox
[    2.450669] hid: raw HID events driver (C) Jiri Kosina
[    2.462143] usbcore: registered new interface driver usbhid
[    2.462147] usbhid: USB HID core driver
[    2.464293] input: VirtualBox USB Tablet as /devices/pci0000:00/0000:00:06.0/usb2/2-1/2-1:1.0/0003:80EE:0021.0001/input/input4
[    2.464413] hid-generic 0003:80EE:0021.0001: input,hidraw0: USB HID v1.10 Mouse [VirtualBox USB Tablet] on usb-0000:00:06.0-1/input0
[    2.832865] PM: Image not found (code -22)
[    3.043343] EXT4-fs (sda1): mounted filesystem with ordered data mode. Quota mode: none.
[    3.110000] Not activating Mandatory Access Control as /sbin/tomoyo-init does not exist.
[    3.263470] systemd[1]: Inserted module 'autofs4'
[    3.297101] systemd[1]: systemd 252.30-1~deb12u2 running in system mode (+PAM +AUDIT +SELINUX +APPARMOR +IMA +SMACK +SECCOMP +GCRYPT -GNUTLS +OPENSSL +ACL +BLKID +CURL +ELFUTILS +FIDO2 +IDN2 -IDN +IPTC +KMOD +LIBCRYPTSETUP +LIBFDISK +PCRE2 -PWQUALITY +P11KIT +QRENCODE +TPM2 +BZIP2 +LZ4 +XZ +ZLIB +ZSTD -BPF_FRAMEWORK -XKBCOMMON +UTMP +SYSVINIT default-hierarchy=unified)
[    3.297107] systemd[1]: Detected virtualization oracle.
[    3.297111] systemd[1]: Detected architecture x86-64.
[    3.300093] systemd[1]: Hostname set to <serveur1>.
[    3.302284] systemd[1]: Invalid DMI field header.
[    3.628050] systemd[1]: Queued start job for default target graphical.target.
[    3.639641] systemd[1]: Created slice system-getty.slice - Slice /system/getty.
[    3.640021] systemd[1]: Created slice system-modprobe.slice - Slice /system/modprobe.
[    3.640318] systemd[1]: Created slice system-systemd\x2dfsck.slice - Slice /system/systemd-fsck.
[    3.640551] systemd[1]: Created slice user.slice - User and Session Slice.
[    3.640634] systemd[1]: Started systemd-ask-password-console.path - Dispatch Password Requests to Console Directory Watch.
[    3.640696] systemd[1]: Started systemd-ask-password-wall.path - Forward Password Requests to Wall Directory Watch.
[    3.640891] systemd[1]: Set up automount proc-sys-fs-binfmt_misc.automount - Arbitrary Executable File Formats File System Automount Point.
[    3.640919] systemd[1]: Expecting device dev-disk-by\x2duuid-269e3730\x2d9ab4\x2d4c1e\x2d9bda\x2de1015c35634d.device - /dev/disk/by-uuid/269e3730-9ab4-4c1e-9bda-e1015c35634d...
[    3.640928] systemd[1]: Expecting device dev-disk-by\x2duuid-563da21f\x2d4d59\x2d4787\x2d8c22\x2df7422e2b9599.device - /dev/disk/by-uuid/563da21f-4d59-4787-8c22-f7422e2b9599...
[    3.640936] systemd[1]: Expecting device dev-disk-by\x2duuid-edd72b9b\x2d9424\x2d4bf1\x2d96ef\x2dbfdc5bb7a147.device - /dev/disk/by-uuid/edd72b9b-9424-4bf1-96ef-bfdc5bb7a147...
[    3.640951] systemd[1]: Reached target cryptsetup.target - Local Encrypted Volumes.
[    3.640973] systemd[1]: Reached target integritysetup.target - Local Integrity Protected Volumes.
[    3.640997] systemd[1]: Reached target paths.target - Path Units.
[    3.641015] systemd[1]: Reached target remote-fs.target - Remote File Systems.
[    3.641031] systemd[1]: Reached target slices.target - Slice Units.
[    3.641061] systemd[1]: Reached target veritysetup.target - Local Verity Protected Volumes.
[    3.641209] systemd[1]: Listening on systemd-fsckd.socket - fsck to fsckd communication Socket.
[    3.641276] systemd[1]: Listening on systemd-initctl.socket - initctl Compatibility Named Pipe.
[    3.641529] systemd[1]: Listening on systemd-journald-audit.socket - Journal Audit Socket.
[    3.641655] systemd[1]: Listening on systemd-journald-dev-log.socket - Journal Socket (/dev/log).
[    3.641808] systemd[1]: Listening on systemd-journald.socket - Journal Socket.
[    3.642867] systemd[1]: Listening on systemd-udevd-control.socket - udev Control Socket.
[    3.643006] systemd[1]: Listening on systemd-udevd-kernel.socket - udev Kernel Socket.
[    3.644012] systemd[1]: Mounting dev-hugepages.mount - Huge Pages File System...
[    3.645078] systemd[1]: Mounting dev-mqueue.mount - POSIX Message Queue File System...
[    3.646103] systemd[1]: Mounting sys-kernel-debug.mount - Kernel Debug File System...
[    3.647800] systemd[1]: Mounting sys-kernel-tracing.mount - Kernel Trace File System...
[    3.660264] systemd[1]: Starting keyboard-setup.service - Set the console keyboard layout...
[    3.661582] systemd[1]: Starting kmod-static-nodes.service - Create List of Static Device Nodes...
[    3.671032] systemd[1]: Starting modprobe@configfs.service - Load Kernel Module configfs...
[    3.686449] systemd[1]: Starting modprobe@dm_mod.service - Load Kernel Module dm_mod...
[    3.689133] systemd[1]: Starting modprobe@drm.service - Load Kernel Module drm...
[    3.705598] systemd[1]: Starting modprobe@efi_pstore.service - Load Kernel Module efi_pstore...
[    3.719162] systemd[1]: Starting modprobe@fuse.service - Load Kernel Module fuse...
[    3.727463] systemd[1]: Starting modprobe@loop.service - Load Kernel Module loop...
[    3.727726] systemd[1]: systemd-fsck-root.service - File System Check on Root Device was skipped because of an unmet condition check (ConditionPathExists=!/run/initramfs/fsck-root).
[    3.755369] systemd[1]: Starting systemd-journald.service - Journal Service...
[    3.767985] systemd[1]: Starting systemd-modules-load.service - Load Kernel Modules...
[    3.774322] device-mapper: core: CONFIG_IMA_DISABLE_HTABLE is disabled. Duplicate IMA measurements will not be recorded in the IMA log.
[    3.774357] device-mapper: uevent: version 1.0.3
[    3.786798] systemd[1]: Starting systemd-remount-fs.service - Remount Root and Kernel File Systems...
[    3.809456] systemd[1]: Starting systemd-udev-trigger.service - Coldplug All udev Devices...
[    3.817562] device-mapper: ioctl: 4.47.0-ioctl (2022-07-28) initialised: dm-devel@redhat.com
[    3.819035] loop: module loaded
[    3.832029] systemd[1]: Mounted dev-hugepages.mount - Huge Pages File System.
[    3.832146] systemd[1]: Mounted dev-mqueue.mount - POSIX Message Queue File System.
[    3.832242] systemd[1]: Mounted sys-kernel-debug.mount - Kernel Debug File System.
[    3.832338] systemd[1]: Mounted sys-kernel-tracing.mount - Kernel Trace File System.
[    3.832575] systemd[1]: Finished kmod-static-nodes.service - Create List of Static Device Nodes.
[    3.832921] systemd[1]: modprobe@configfs.service: Deactivated successfully.
[    3.833470] systemd[1]: Finished modprobe@configfs.service - Load Kernel Module configfs.
[    3.833780] systemd[1]: modprobe@dm_mod.service: Deactivated successfully.
[    3.839120] systemd[1]: Finished modprobe@dm_mod.service - Load Kernel Module dm_mod.
[    3.839525] systemd[1]: modprobe@efi_pstore.service: Deactivated successfully.
[    3.839680] systemd[1]: Finished modprobe@efi_pstore.service - Load Kernel Module efi_pstore.
[    3.842553] fuse: init (API version 7.37)
[    3.851628] systemd[1]: modprobe@loop.service: Deactivated successfully.
[    3.851812] systemd[1]: Finished modprobe@loop.service - Load Kernel Module loop.
[    3.852151] systemd[1]: Finished systemd-modules-load.service - Load Kernel Modules.
[    3.883906] systemd[1]: Mounting sys-kernel-config.mount - Kernel Configuration File System...
[    3.884358] systemd[1]: systemd-repart.service - Repartition Root Disk was skipped because no trigger condition checks were met.
[    3.890350] systemd[1]: Starting systemd-sysctl.service - Apply Kernel Variables...
[    3.902391] systemd[1]: modprobe@fuse.service: Deactivated successfully.
[    3.904554] EXT4-fs (sda1): re-mounted. Quota mode: none.
[    3.926341] systemd[1]: Finished modprobe@fuse.service - Load Kernel Module fuse.
[    3.936621] systemd[1]: Finished systemd-remount-fs.service - Remount Root and Kernel File Systems.
[    3.945154] systemd[1]: Mounted sys-kernel-config.mount - Kernel Configuration File System.
[    3.965817] systemd[1]: Mounting sys-fs-fuse-connections.mount - FUSE Control File System...
[    3.965984] systemd[1]: systemd-firstboot.service - First Boot Wizard was skipped because of an unmet condition check (ConditionFirstBoot=yes).
[    3.966197] systemd[1]: systemd-pstore.service - Platform Persistent Storage Archival was skipped because of an unmet condition check (ConditionDirectoryNotEmpty=/sys/fs/pstore).
[    3.967927] systemd[1]: Starting systemd-random-seed.service - Load/Save Random Seed...
[    3.969641] systemd[1]: Starting systemd-sysusers.service - Create System Users...
[    3.985373] systemd[1]: Finished systemd-sysctl.service - Apply Kernel Variables.
[    3.985689] systemd[1]: Mounted sys-fs-fuse-connections.mount - FUSE Control File System.
[    4.007455] ACPI: bus type drm_connector registered
[    4.008669] systemd[1]: modprobe@drm.service: Deactivated successfully.
[    4.008829] systemd[1]: Finished modprobe@drm.service - Load Kernel Module drm.
[    4.032553] systemd[1]: Finished systemd-random-seed.service - Load/Save Random Seed.
[    4.032678] systemd[1]: first-boot-complete.target - First Boot Complete was skipped because of an unmet condition check (ConditionFirstBoot=yes).
[    4.055756] systemd[1]: Finished systemd-sysusers.service - Create System Users.
[    4.085030] systemd[1]: Starting systemd-tmpfiles-setup-dev.service - Create Static Device Nodes in /dev...
[    4.085199] systemd[1]: Started systemd-journald.service - Journal Service.
[    4.449816] sd 2:0:0:0: Attached scsi generic sg0 type 0
[    4.449849] sr 3:0:0:0: Attached scsi generic sg1 type 5
[    4.489877] input: Power Button as /devices/LNXSYSTM:00/LNXPWRBN:00/input/input5
[    4.491855] ACPI: AC: AC Adapter [AC] (off-line)
[    4.504216] ACPI: button: Power Button [PWRF]
[    4.504353] input: Sleep Button as /devices/LNXSYSTM:00/LNXSLPBN:00/input/input6
[    4.505111] ACPI: button: Sleep Button [SLPF]
[    4.569050] input: PC Speaker as /devices/platform/pcspkr/input/input7
[    4.588148] vboxguest: host-version: 7.1.2r164945 0x8000000f
[    4.649262] vbg_heartbeat_init: Setting up heartbeat to trigger every 2000 milliseconds
[    4.649579] input: VirtualBox mouse integration as /devices/pci0000:00/0000:00:04.0/input/input8
[    4.876403] RAPL PMU: API unit is 2^-32 Joules, 0 fixed counters, 10737418240 ms ovfl timer
[    4.896354] cryptd: max_cpu_qlen set to 1000
[    4.943832] AVX2 version of gcm_enc/dec engaged.
[    4.943859] AES CTR mode by8 optimization enabled
[    4.952730] vmwgfx 0000:00:02.0: vgaarb: deactivate vga console
[    5.007746] Adding 6321148k swap on /dev/sda4.  Priority:-2 extents:1 across:6321148k FS
[    5.018042] Console: switching to colour dummy device 80x25
[    5.019382] vmwgfx 0000:00:02.0: [drm] FIFO at 0x00000000f0000000 size is 2048 kiB
[    5.019396] vmwgfx 0000:00:02.0: [drm] VRAM at 0x00000000e0000000 size is 16384 kiB
[    5.019473] vmwgfx 0000:00:02.0: [drm] Running on SVGA version 2.
[    5.019512] vmwgfx 0000:00:02.0: [drm] Capabilities: rect copy, cursor, cursor bypass, cursor bypass 2, alpha cursor, extended fifo, pitchlock, irq mask, gmr, traces, gmr2, screen object 2, command buffers, 
[    5.019515] vmwgfx 0000:00:02.0: [drm] DMA map mode: Caching DMA mappings.
[    5.019817] vmwgfx 0000:00:02.0: [drm] Legacy memory limits: VRAM = 16384 kB, FIFO = 2048 kB, surface = 507904 kB
[    5.019820] vmwgfx 0000:00:02.0: [drm] MOB limits: max mob size = 0 kB, max mob pages = 0
[    5.019822] vmwgfx 0000:00:02.0: [drm] Max GMR ids is 8192
[    5.019823] vmwgfx 0000:00:02.0: [drm] Max number of GMR pages is 1048576
[    5.019825] vmwgfx 0000:00:02.0: [drm] Maximum display memory size is 16384 kiB
[    5.028483] vmwgfx 0000:00:02.0: [drm] Screen Object display unit initialized
[    5.033355] vmwgfx 0000:00:02.0: [drm] Fifo max 0x00200000 min 0x00001000 cap 0x00000355
[    5.059243] vmwgfx 0000:00:02.0: [drm] Using command buffers with DMA pool.
[    5.059257] vmwgfx 0000:00:02.0: [drm] Available shader model: Legacy.
[    5.059281] [drm:vmw_host_printf [vmwgfx]] *ERROR* Failed to send host log message.
[    5.078239] [drm] Initialized vmwgfx 2.20.0 20211206 for 0000:00:02.0 on minor 0
[    5.084743] fbcon: vmwgfxdrmfb (fb0) is primary device
[    5.119025] Console: switching to colour frame buffer device 160x50
[    5.179405] vmwgfx 0000:00:02.0: [drm] fb0: vmwgfxdrmfb frame buffer device
[    5.321719] snd_intel8x0 0000:00:05.0: allow list rate for 1028:0177 is 48000
[    5.377174] EXT4-fs (sda3): mounted filesystem with ordered data mode. Quota mode: none.
[    5.432264] systemd-journald[215]: Received client request to flush runtime journal.
[    5.556420] EXT4-fs (sda2): mounted filesystem with ordered data mode. Quota mode: none.
[    5.750339] audit: type=1400 audit(1729026982.323:2): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lsb_release" pid=433 comm="apparmor_parser"
[    5.779625] audit: type=1400 audit(1729026982.355:3): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe" pid=436 comm="apparmor_parser"
[    5.779631] audit: type=1400 audit(1729026982.355:4): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe//kmod" pid=436 comm="apparmor_parser"
[    5.827372] audit: type=1400 audit(1729026982.403:5): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=437 comm="apparmor_parser"
[    5.827378] audit: type=1400 audit(1729026982.403:6): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-helper" pid=437 comm="apparmor_parser"
[    5.827380] audit: type=1400 audit(1729026982.403:7): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=437 comm="apparmor_parser"
[    5.827382] audit: type=1400 audit(1729026982.403:8): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/{,usr/}sbin/dhclient" pid=437 comm="apparmor_parser"
[    5.850584] audit: type=1400 audit(1729026982.423:9): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/bin/man" pid=438 comm="apparmor_parser"
[    5.850590] audit: type=1400 audit(1729026982.423:10): apparmor="STATUS" operation="profile_load" profile="unconfined" name="man_filter" pid=438 comm="apparmor_parser"
[    5.850592] audit: type=1400 audit(1729026982.423:11): apparmor="STATUS" operation="profile_load" profile="unconfined" name="man_groff" pid=438 comm="apparmor_parser"
[    6.178113] e1000: enp0s3 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
[    6.179776] IPv6: ADDRCONF(NETDEV_CHANGE): enp0s3: link becomes ready
```


Pour savoir notre carte reseaux ou le modèle de processeur linux, il faut chercher parmis c'est lignes les informations qui nous interesses par exemple:

**Processeur linux:**

```
[    0.000000] Linux version 6.1.0-25-amd64 (debian-kernel@lists.debian.org) (gcc-12 (Debian 12.2.0-14) 12.2.0, GNU ld (GNU Binutils for Debian) 2.40) #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26)
```

**Carte Reseaux :**

```
[    2.133167] e1000 0000:00:03.0 eth0: (PCI:33MHz:32-bit) 08:00:27:12:3e:86
[    2.133176] e1000 0000:00:03.0 eth0: Intel(R) PRO/1000 Network Connection
[    2.231468] ata4: SATA link up 3.0 Gbps (SStatus 123 SControl 300)
```

## Source

https://www.debian.org/

https://linuxfr.org/forums/

https://forum.ubuntu-fr.org/
