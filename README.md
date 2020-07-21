# WordPressRedTeam_BlueTeam

The network depicted below was used to test exploits and determine vulnerabilities within the WordPress servers.
![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/Topology%20Final%20Project.png)
Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

# Exposed Services
$ nmap -sC -sV 192.168.1.110

![TODO: Update the path with the name of your diagram](https://github.com/joshgarlandreese/WordPressRedTeam_BlueTeam/blob/master/NMAP%20final%20Project.png)
This scan identifies the services below as potential points of entry:

Target 1
Port 22 / SSH
Port 80 / http
Port 111 / rpcbind
