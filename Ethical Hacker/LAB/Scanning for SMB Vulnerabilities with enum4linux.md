_______________________________________________

Scanne de Vulnérabilités du SMB avec enum4linux
_______________________________________________

Définition de quelques termes

- SID (Security Identifier)
  - Le SID  est un identifiant unique qui désigne un utilisateur, un groupe ou un domaine sur windows
  - Il est composé de plusieurs parties dont le RID
  - Format : S-1-5-21-3623811015-3361044348-30300820-500 dont le 500 représente le RID

- RID (Relative Identifier)
  - C'est un composant du SID qui permet d'identifier un utilisateur ou des objets dans un domaine ou un système locale  
  - 500 : Administrateur du domaine ou de l'ordinateur local (compte administrateur par défaut).
  - 501 : Invité du domaine ou de l'ordinateur local (compte invité par défaut).
  - 1000 et au-delà : Comptes d'utilisateurs créés sur le système.
  - Lorsqu'un nouveau compte utilisateur ou groupe est créé sur un domaine ou un système, un nouveau RID est généré et ajouté au SID du compte
  - Connaître le RID de comptes spécifiques (comme le RID 500 pour l'administrateur) peut être exploité pour cibler directement ces comptess.

## Utilisation de Nmap pour déterminer les serveurs SMB

### Scanne du réseau virtuel 172.17.0.0/24

```bash
nmap -sN 172.17.0.0/24
Starting Nmap 7.94 ( https://nmap.org ) at 2024-08-20 16:12 UTC
Nmap scan report for metasploitable.vm (172.17.0.2)
Host is up (0.000023s latency).
Not shown: 981 closed tcp ports (reset)
PORT     STATE         SERVICE
21/tcp   open|filtered ftp
22/tcp   open|filtered ssh
23/tcp   open|filtered telnet
25/tcp   open|filtered smtp
80/tcp   open|filtered http
111/tcp  open|filtered rpcbind
139/tcp  open|filtered netbios-ssn
445/tcp  open|filtered microsoft-ds
512/tcp  open|filtered exec
513/tcp  open|filtered login
514/tcp  open|filtered shell
1099/tcp open|filtered rmiregistry
1524/tcp open|filtered ingreslock
2121/tcp open|filtered ccproxy-ftp
3306/tcp open|filtered mysql
5432/tcp open|filtered postgresql
6667/tcp open|filtered irc
8009/tcp open|filtered ajp13
8180/tcp open|filtered unknown
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap scan report for 172.17.0.1
Host is up (0.000039s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE         SERVICE
22/tcp open|filtered ssh

Nmap done: 256 IP addresses (2 hosts up) scanned in 4.74 seconds
```

- Résultats :
  - On remarque que le port 139 & 445 son ouverts sur l'hôte 172.17.0.2

### Scanne du sous réseau 10.6.6.0/24

```bash
┌──(root㉿Kali)-[/home/kali]
└─# nmap -sN 10.6.6.0/24
Starting Nmap 7.94 ( https://nmap.org ) at 2024-08-20 16:13 UTC
Nmap scan report for webgoat.vm (10.6.6.11)
Host is up (0.000033s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE         SERVICE
8080/tcp open|filtered http-proxy
8888/tcp open|filtered sun-answerbook
9001/tcp open|filtered tor-orport
MAC Address: 02:42:0A:06:06:0B (Unknown)

Nmap scan report for juice-shop.vm (10.6.6.12)
Host is up (0.000023s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE         SERVICE
3000/tcp open|filtered ppp
MAC Address: 02:42:0A:06:06:0C (Unknown)

Nmap scan report for dvwa.vm (10.6.6.13)
Host is up (0.000042s latency).
All 1000 scanned ports on dvwa.vm (10.6.6.13) are in ignored states.
Not shown: 1000 closed tcp ports (reset)
MAC Address: 02:42:0A:06:06:0D (Unknown)

Nmap scan report for mutillidae.vm (10.6.6.14)
Host is up (0.000042s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE         SERVICE
80/tcp   open|filtered http
3306/tcp open|filtered mysql
MAC Address: 02:42:0A:06:06:0E (Unknown)

Nmap scan report for gravemind.vm (10.6.6.23)
Host is up (0.000042s latency).
Not shown: 994 closed tcp ports (reset)
PORT    STATE         SERVICE
21/tcp  open|filtered ftp
22/tcp  open|filtered ssh
53/tcp  open|filtered domain
80/tcp  open|filtered http
139/tcp open|filtered netbios-ssn
445/tcp open|filtered microsoft-ds
MAC Address: 02:42:0A:06:06:17 (Unknown)

Nmap scan report for 10.6.6.100
Host is up (0.000029s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE         SERVICE
80/tcp open|filtered http
MAC Address: 02:42:0A:06:06:64 (Unknown)

Nmap scan report for 10.6.6.1
Host is up (0.000029s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE         SERVICE
22/tcp open|filtered ssh

Nmap done: 256 IP addresses (7 hosts up) scanned in 4.93 seconds
```

- Résultats
  - On constate que sur l'hôte 10.6.6.23 les ports 139 (NetBIOS) & 445 (SMB) sont ouverts. Ce qui fait de cet hôte une cible potentiel

## Utilisation de enum4linux pour l'énumération des utilisateurs et partages de fichiers

### Utilisation de enum4linux sur la cible 172.17.0.2

#### Déterminons les utilisateurs présents sur la cible avec enum4linux - u

```bash
┌──(root㉿Kali)-[/home/kali]
└─# enum4linux -U 172.17.0.2
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Aug 20 16:32:46 2024

 =========================================( Target Information )=========================================

Target ........... 172.17.0.2
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 =============================( Enumerating Workgroup/Domain on 172.17.0.2 )=============================


[+] Got domain/workgroup name: WORKGROUP


 ====================================( Session Check on 172.17.0.2 )====================================


[+] Server 172.17.0.2 allows sessions using username '', password ''


 =================================( Getting domain SID for 172.17.0.2 )=================================

Domain Name: WORKGROUP
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup


 ========================================( Users on 172.17.0.2 )========================================

index: 0x1 RID: 0x3f2 acb: 0x00000011 Account: games    Name: games     Desc: (null)
index: 0x2 RID: 0x1f5 acb: 0x00000011 Account: nobody   Name: nobody    Desc: (null)
index: 0x3 RID: 0x4ba acb: 0x00000011 Account: bind     Name: (null)    Desc: (null)
index: 0x4 RID: 0x402 acb: 0x00000011 Account: proxy    Name: proxy     Desc: (null)
index: 0x5 RID: 0x4b4 acb: 0x00000011 Account: syslog   Name: (null)    Desc: (null)
index: 0x6 RID: 0xbba acb: 0x00000010 Account: user     Name: just a user,111,, Desc: (null)
index: 0x7 RID: 0x42a acb: 0x00000011 Account: www-data Name: www-data  Desc: (null)
index: 0x8 RID: 0x3e8 acb: 0x00000011 Account: root     Name: root      Desc: (null)
index: 0x9 RID: 0x3fa acb: 0x00000011 Account: news     Name: news      Desc: (null)
index: 0xa RID: 0x4c0 acb: 0x00000011 Account: postgres Name: PostgreSQL administrator,,,      Desc: (null)
index: 0xb RID: 0x3ec acb: 0x00000011 Account: bin      Name: bin       Desc: (null)
index: 0xc RID: 0x3f8 acb: 0x00000011 Account: mail     Name: mail      Desc: (null)
index: 0xd RID: 0x4c6 acb: 0x00000011 Account: distccd  Name: (null)    Desc: (null)
index: 0xe RID: 0x4ca acb: 0x00000011 Account: proftpd  Name: (null)    Desc: (null)
index: 0xf RID: 0x4b2 acb: 0x00000011 Account: dhcp     Name: (null)    Desc: (null)
index: 0x10 RID: 0x3ea acb: 0x00000011 Account: daemon  Name: daemon    Desc: (null)
index: 0x11 RID: 0x4b8 acb: 0x00000011 Account: sshd    Name: (null)    Desc: (null)
index: 0x12 RID: 0x3f4 acb: 0x00000011 Account: man     Name: man       Desc: (null)
index: 0x13 RID: 0x3f6 acb: 0x00000011 Account: lp      Name: lp        Desc: (null)
index: 0x14 RID: 0x4c2 acb: 0x00000011 Account: mysql   Name: MySQL Server,,,   Desc: (null)
index: 0x15 RID: 0x43a acb: 0x00000011 Account: gnats   Name: Gnats Bug-Reporting System (admin)       Desc: (null)
index: 0x16 RID: 0x4b0 acb: 0x00000011 Account: libuuid Name: (null)    Desc: (null)
index: 0x17 RID: 0x42c acb: 0x00000011 Account: backup  Name: backup    Desc: (null)
index: 0x18 RID: 0xbb8 acb: 0x00000010 Account: msfadmin        Name: msfadmin,,,     Desc: (null)
index: 0x19 RID: 0x4c8 acb: 0x00000011 Account: telnetd Name: (null)    Desc: (null)
index: 0x1a RID: 0x3ee acb: 0x00000011 Account: sys     Name: sys       Desc: (null)
index: 0x1b RID: 0x4b6 acb: 0x00000011 Account: klog    Name: (null)    Desc: (null)
index: 0x1c RID: 0x4bc acb: 0x00000011 Account: postfix Name: (null)    Desc: (null)
index: 0x1d RID: 0xbbc acb: 0x00000011 Account: service Name: ,,,       Desc: (null)
index: 0x1e RID: 0x434 acb: 0x00000011 Account: list    Name: Mailing List Manager    Desc: (null)
index: 0x1f RID: 0x436 acb: 0x00000011 Account: irc     Name: ircd      Desc: (null)
index: 0x20 RID: 0x4be acb: 0x00000011 Account: ftp     Name: (null)    Desc: (null)
index: 0x21 RID: 0x4c4 acb: 0x00000011 Account: tomcat55        Name: (null)    Desc: (null)
index: 0x22 RID: 0x3f0 acb: 0x00000011 Account: sync    Name: sync      Desc: (null)
index: 0x23 RID: 0x3fc acb: 0x00000011 Account: uucp    Name: uucp      Desc: (null)

user:[games] rid:[0x3f2]
user:[nobody] rid:[0x1f5]
user:[bind] rid:[0x4ba]
user:[proxy] rid:[0x402]
user:[syslog] rid:[0x4b4]
user:[user] rid:[0xbba]
user:[www-data] rid:[0x42a]
user:[root] rid:[0x3e8]
user:[news] rid:[0x3fa]
user:[postgres] rid:[0x4c0]
user:[bin] rid:[0x3ec]
user:[mail] rid:[0x3f8]
user:[distccd] rid:[0x4c6]
user:[proftpd] rid:[0x4ca]
user:[dhcp] rid:[0x4b2]
user:[daemon] rid:[0x3ea]
user:[sshd] rid:[0x4b8]
user:[man] rid:[0x3f4]
user:[lp] rid:[0x3f6]
user:[mysql] rid:[0x4c2]
user:[gnats] rid:[0x43a]
user:[libuuid] rid:[0x4b0]
user:[backup] rid:[0x42c]
user:[msfadmin] rid:[0xbb8]
user:[telnetd] rid:[0x4c8]
user:[sys] rid:[0x3ee]
user:[klog] rid:[0x4b6]
user:[postfix] rid:[0x4bc]
user:[service] rid:[0xbbc]
user:[list] rid:[0x434]
user:[irc] rid:[0x436]
user:[ftp] rid:[0x4be]
user:[tomcat55] rid:[0x4c4]
user:[sync] rid:[0x3f0]
user:[uucp] rid:[0x3fc]
enum4linux complete on Tue Aug 20 16:32:48 2024
```

#### Déterminons les partages réseaux présents avec enum4linux -S

```bash
┌──(root㉿Kali)-[/home/kali]
└─# enum4linux -Sv 172.17.0.2

[V] Dependent program "nmblookup" found in /usr/bin/nmblookup


[V] Dependent program "net" found in /usr/bin/net


[V] Dependent program "rpcclient" found in /usr/bin/rpcclient


[V] Dependent program "smbclient" found in /usr/bin/smbclient


[V] Dependent program "polenum" found in /usr/bin/polenum


[V] Dependent program "ldapsearch" found in /usr/bin/ldapsearch

Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Aug 20 16:41:49 2024

 =========================================( Target Information )=========================================

Target ........... 172.17.0.2
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 =============================( Enumerating Workgroup/Domain on 172.17.0.2 )=============================


[V] Attempting to get domain name with command: nmblookup -A '172.17.0.2'


[+] Got domain/workgroup name: WORKGROUP


 ====================================( Session Check on 172.17.0.2 )====================================


[V] Attempting to make null session using command: smbclient -W 'WORKGROUP' //'172.17.0.2'/ipc$ -U''%'' -c 'help' 2>&1


[+] Server 172.17.0.2 allows sessions using username '', password ''


 =================================( Getting domain SID for 172.17.0.2 )=================================


[V] Attempting to get domain SID with command: rpcclient -W 'WORKGROUP' -U''%'' 172.17.0.2 -c 'lsaquery' 2>&1

Domain Name: WORKGROUP
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup


 ==================================( Share Enumeration on 172.17.0.2 )==================================


[V] Attempting to get share list using authentication


        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk
        IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            METASPLOITABLE

[+] Attempting to map shares on 172.17.0.2


[V] Attempting map to share //172.17.0.2/print$ with command: smbclient -W 'WORKGROUP' //'172.17.0.2'/'print$' -U''%'' -c dir 2>&1

//172.17.0.2/print$     Mapping: DENIED Listing: N/A Writing: N/A

[V] Attempting map to share //172.17.0.2/tmp with command: smbclient -W 'WORKGROUP' //'172.17.0.2'/'tmp' -U''%'' -c dir 2>&1

//172.17.0.2/tmp        Mapping: OK Listing: OK Writing: N/A

[V] Attempting map to share //172.17.0.2/opt with command: smbclient -W 'WORKGROUP' //'172.17.0.2'/'opt' -U''%'' -c dir 2>&1

//172.17.0.2/opt        Mapping: DENIED Listing: N/A Writing: N/A

[V] Attempting map to share //172.17.0.2/IPC$ with command: smbclient -W 'WORKGROUP' //'172.17.0.2'/'IPC$' -U''%'' -c dir 2>&1


[E] Can't understand response:

NT_STATUS_NETWORK_ACCESS_DENIED listing \*
//172.17.0.2/IPC$       Mapping: N/A Listing: N/A Writing: N/A

[V] Attempting map to share //172.17.0.2/ADMIN$ with command: smbclient -W 'WORKGROUP' //'172.17.0.2'/'ADMIN$' -U''%'' -c dir 2>&1

//172.17.0.2/ADMIN$     Mapping: DENIED Listing: N/A Writing: N/A
enum4linux complete on Tue Aug 20 16:41:51 2024

```

- Résultats :

  - Nous avons 5 fichier partagés dont 2 cachés

#### Déterminons les politiques de mots de passe mise en place avec enum4linux -P

```bash
┌──(root㉿Kali)-[/home/kali]
└─# enum4linux -P 172.17.0.2
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Aug 20 16:49:27 2024

 =========================================( Target Information )=========================================

Target ........... 172.17.0.2
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 =============================( Enumerating Workgroup/Domain on 172.17.0.2 )=============================


[+] Got domain/workgroup name: WORKGROUP


 ====================================( Session Check on 172.17.0.2 )====================================


[+] Server 172.17.0.2 allows sessions using username '', password ''


 =================================( Getting domain SID for 172.17.0.2 )=================================

Domain Name: WORKGROUP
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup


 =============================( Password Policy Information for 172.17.0.2 )=============================



[+] Attaching to 172.17.0.2 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

        [+] METASPLOITABLE
        [+] Builtin

[+] Password Info for Domain: METASPLOITABLE

        [+] Minimum password length: 5
        [+] Password history length: None
        [+] Maximum password age: Not Set
        [+] Password Complexity Flags: 000000

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 0

        [+] Minimum password age: None
        [+] Reset Account Lockout Counter: 30 minutes
        [+] Locked Account Duration: 30 minutes
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: Not Set



[+] Retieved partial password policy with rpcclient:


Password Complexity: Disabled
Minimum Password Length: 0

enum4linux complete on Tue Aug 20 16:49:30 2024

```

- Résultats :
  - Longueur minimal du password = 5
  - Il n'existe pas d'account lockout
  - Niveau de sécurité bas

#### Simple énumération sur la cible 10.6.6.23 avec enum4linux -a

```bash
┌──(root㉿Kali)-[/home/kali]
└─# enum4linux -a 10.6.6.23
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Aug 20 16:55:46 2024

 =========================================( Target Information )=========================================

Target ........... 10.6.6.23
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 =============================( Enumerating Workgroup/Domain on 10.6.6.23 )=============================


[E] Can't find workgroup/domain



 =================================( Nbtstat Information for 10.6.6.23 )=================================

Looking up status of 10.6.6.23
No reply from 10.6.6.23

 =====================================( Session Check on 10.6.6.23 )=====================================


[+] Server 10.6.6.23 allows sessions using username '', password ''


 ==================================( Getting domain SID for 10.6.6.23 )==================================

Domain Name: WORKGROUP
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup


 ====================================( OS information on 10.6.6.23 )====================================


[E] Can't get OS info with smbclient


[+] Got OS info for 10.6.6.23 from srvinfo:
        GRAVEMIND      Wk Sv PrQ Unx NT SNT Samba 4.9.5-Debian
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03


 =========================================( Users on 10.6.6.23 )=========================================

index: 0x1 RID: 0x3e8 acb: 0x00000015 Account: masterchief      Name:   Desc:
index: 0x2 RID: 0x3e9 acb: 0x00000015 Account: arbiter  Name:   Desc:

user:[masterchief] rid:[0x3e8]
user:[arbiter] rid:[0x3e9]

 ===================================( Share Enumeration on 10.6.6.23 )===================================


        Sharename       Type      Comment
        ---------       ----      -------
        homes           Disk      All home directories
        workfiles       Disk      Confidential Workfiles
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------

[+] Attempting to map shares on 10.6.6.23


[E] Can't understand response:

tree connect failed: NT_STATUS_BAD_NETWORK_NAME
//10.6.6.23/homes       Mapping: N/A Listing: N/A Writing: N/A
//10.6.6.23/workfiles   Mapping: OK Listing: OK Writing: N/A
//10.6.6.23/print$      Mapping: OK Listing: OK Writing: N/A

[E] Can't understand response:

NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
//10.6.6.23/IPC$        Mapping: N/A Listing: N/A Writing: N/A

 =============================( Password Policy Information for 10.6.6.23 )=============================



[+] Attaching to 10.6.6.23 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

        [+] GRAVEMIND
        [+] Builtin

[+] Password Info for Domain: GRAVEMIND

        [+] Minimum password length: 5
        [+] Password history length: None
        [+] Maximum password age: 37 days 6 hours 21 minutes
        [+] Password Complexity Flags: 000000

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 0

        [+] Minimum password age: None
        [+] Reset Account Lockout Counter: 30 minutes
        [+] Locked Account Duration: 30 minutes
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: 37 days 6 hours 21 minutes



[+] Retieved partial password policy with rpcclient:


Password Complexity: Disabled
Minimum Password Length: 5


 ========================================( Groups on 10.6.6.23 )========================================


[+] Getting builtin groups:


[+]  Getting builtin group memberships:


[+]  Getting local groups:


[+]  Getting local group memberships:


[+]  Getting domain groups:


[+]  Getting domain group memberships:


 ====================( Users on 10.6.6.23 via RID cycling (RIDS: 500-550,1000-1050) )====================


[I] Found new SID:
S-1-22-1

[I] Found new SID:
S-1-5-32

[I] Found new SID:
S-1-5-32

[I] Found new SID:
S-1-5-32

[I] Found new SID:
S-1-5-32

[+] Enumerating users using SID S-1-5-32 and logon username '', password ''

S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

[+] Enumerating users using SID S-1-5-21-3080196717-3701805971-2094628062 and logon username '', password ''

S-1-5-21-3080196717-3701805971-2094628062-501 GRAVEMIND\nobody (Local User)
S-1-5-21-3080196717-3701805971-2094628062-513 GRAVEMIND\None (Domain Group)
S-1-5-21-3080196717-3701805971-2094628062-1000 GRAVEMIND\masterchief (Local User)
S-1-5-21-3080196717-3701805971-2094628062-1001 GRAVEMIND\arbiter (Local User)

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\masterchief (Local User)
S-1-22-1-1001 Unix User\arbiter (Local User)
S-1-22-1-1002 Unix User\labuser (Local User)

 =================================( Getting printer info for 10.6.6.23 )=================================

No printers returned.


enum4linux complete on Tue Aug 20 16:57:47 2024

```

- Résultats :
 
  - Nous constatons qu'il existe 3 utilisateurs et 7 groups
  - Il existe 3 partages et IPC$ est un partages propice au serveur pour ses services

## Utilisation de smbclient pour transférer les fichiers entre systèmes

- ```Smbclient``` est un composant de samba qui permet de stocker et récupérer les fichiers un peu comme FTP

- Création d'un fichier sur la machine attaquante

```bash
┌──(root㉿Kali)-[/home/kali]
└─# cat >> batfile.txt
C'est un fichier Malveillant

```

- listes des partages sur la cible

```bash
┌──(root㉿Kali)-[/home/kali]
└─# smbclient -L //172.17.0.2/
Password for [WORKGROUP\root]:
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk
        IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            METASPLOITABLE

```

- Connection sur le partage tmp grâce à la connection Anonyme

```bash
┌──(root㉿kali)-[/home/kali]

└─# smbclient //172.17.0.2/tmp

Password for [WORKGROUPkali]: <Press enter>

smb: >
```

- Chargement du fichier malveillant sur la cible

```bash
put local-file-name remote-file-name

smb: > put badfile.txt badfile.txt

Putting file badfile.txt as badfile.txt (19.5 kb/s) (average 19.5 kb/s)

```

- Vérification et quitter

```bash
┌──(root㉿Kali)-[/home/kali]
└─# smbclient //172.17.0.2/tmp
Password for [WORKGROUP\root]:
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Aug 20 17:18:00 2024
  ..                                 DR        0  Mon Aug 14 09:39:59 2023
  .X11-unix                          DH        0  Mon Aug 14 09:35:14 2023
  .ICE-unix                          DH        0  Sun Jan 28 02:08:08 2018
  .X0-lock                           HR       11  Mon Aug 14 09:35:14 2023
  gconfd-msfadmin                    DR        0  Sun Aug 18 10:25:32 2024
  orbit-msfadmin                     DR        0  Sun Aug 18 10:25:32 2024
  715.jsvc_up                         R        0  Fri Aug 16 15:36:56 2024
  683.jsvc_up                         R        0  Thu Aug 15 08:35:51 2024
  693.jsvc_up                         R        0  Tue Aug 13 14:21:10 2024
  695.jsvc_up                         R        0  Thu Aug 15 16:57:00 2024
  682.jsvc_up                         R        0  Mon Aug 14 09:35:26 2023
  694.jsvc_up                         R        0  Thu Aug 15 17:42:20 2024
  705.jsvc_up                         R        0  Mon Aug 19 11:10:55 2024
  826.jsvc_up                         R        0  Sun Jan 28 06:08:40 2018
  810.jsvc_up                         R        0  Sun Jan 28 02:54:31 2018
  1582.jsvc_up                        R        0  Sun Jan 28 03:01:49 2018
  1823.jsvc_up                        R        0  Sun Jan 28 01:57:44 2018

                38497656 blocks of size 1024. 8588244 blocks available
smb: \> put batfile.txt batfile.txt
putting file batfile.txt as \batfile.txt (0.0 kb/s) (average 0.0 kb/s)
smb: \> dir
  .                                   D        0  Tue Aug 20 17:18:58 2024
  ..                                 DR        0  Mon Aug 14 09:39:59 2023
  .X11-unix                          DH        0  Mon Aug 14 09:35:14 2023
  .ICE-unix                          DH        0  Sun Jan 28 02:08:08 2018
  .X0-lock                           HR       11  Mon Aug 14 09:35:14 2023
  gconfd-msfadmin                    DR        0  Sun Aug 18 10:25:32 2024
  orbit-msfadmin                     DR        0  Sun Aug 18 10:25:32 2024
  batfile.txt                         A        0  Tue Aug 20 17:18:58 2024
  715.jsvc_up                         R        0  Fri Aug 16 15:36:56 2024
  683.jsvc_up                         R        0  Thu Aug 15 08:35:51 2024
  693.jsvc_up                         R        0  Tue Aug 13 14:21:10 2024
  695.jsvc_up                         R        0  Thu Aug 15 16:57:00 2024
  682.jsvc_up                         R        0  Mon Aug 14 09:35:26 2023
  694.jsvc_up                         R        0  Thu Aug 15 17:42:20 2024
  705.jsvc_up                         R        0  Mon Aug 19 11:10:55 2024
  826.jsvc_up                         R        0  Sun Jan 28 06:08:40 2018
  810.jsvc_up                         R        0  Sun Jan 28 02:54:31 2018
  1582.jsvc_up                        R        0  Sun Jan 28 03:01:49 2018
  1823.jsvc_up                        R        0  Sun Jan 28 01:57:44 2018

                38497656 blocks of size 1024. 8588240 blocks available

```
