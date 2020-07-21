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

