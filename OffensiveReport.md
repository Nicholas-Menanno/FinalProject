# Red Team: Summary of Operations

## Exposed Services

The Nmap scan results show the following results usingthe command:
nmap -v -sV 192.168.1.110

This showed the following services running (note only open ports are listed);


| Port 	| Service     	| Version              	|
|------	|-------------	|----------------------	|
| 22   	| SSH         	| OpenSSH 6.7p1 Debian 	|
| 80   	| HTTP        	| Apache httpd 2.4.10  	|
| 111  	| rcpbind     	| 2-4                  	|
| 139  	| Netbios-ssn 	| Samba smbd 3.X - 4.X 	|
| 445  	| Netbios-ssn 	| Samba smbd 3.X - 4.X 	|

As well as the OS details;
MAC Address: 00:15:5D:00:04:10
Service Info: Host: TARGET1; OS: Linux; cpe:/o:linux\_kernal

On Target 1 there was a few different ways to potentially exploit the machine based on the services running above. There are various exploit modules from msf available to use for the Samba smbd service as well as a potential exploit with rpcbind that would cause a denial of service, which in this case the latter wasn't desirable.

After finding port 80 open I decided to investigate the IP by trying it in Firefox which led to the Raven Security website. After investigating further I discovered a blog section which was hosted on wordpress. Using a WPScan I was able to find two usernames which were steven and michael with the WPScan enumerate user option.

command used: wpscan -v --enumerate vp u --url 192.168.1.110/wordpress/



While im not sure if this is a exploit or just a poorly configured Wordpress deployment it lead to more information about the users. Realizing port 22 was open I decided to try the usernames with some basic passwords to see if I could login. After a few attempts I found that michael:michael worked and gave me an SSH connection to Target 1. If this method didnt work I would attempt to use some method to exploit the Samba service but it wasn't required in this scenario due to poor password security. It would also be possible to attempt to bruteforce the SSH password with the Hydra tool.

command used: ssh michael@192.168.1.110

## CVE Details;

CVE-2017-7494 Samba since version 3.5.0 and before 4.6.4, 4.5.10 and 4.4.14 is vulnerable to remote code execution vulnerability, allowing a malicious client to upload a shared library to a writable share, and then cause the server to load and execute it.
Base Score:  9.8 CRITICAL

CVE-2017-8779 rpcbind through 0.2.4, LIBTIRPC through 1.0.1 and 1.0.2-rc through 1.0.2-rc3, and NTIRPC through 1.4.3 do not consider the maximum RPC data size during memory allocation for XDR strings, which allows remote attackers to cause a denial of service (memory consumption with no subsequent free) via a crafted UDP packet to port 111, aka rpcbomb.
Base Score:  7.5 HIGH

CVE-2017-5487 wp-includes/rest-api/endpoints/class-wp-rest-users-controller.php in the REST API implementation in WordPress 4.7 before 4.7.1 does not properly restrict listings of post authors, which allows remote attackers to obtain sensitive information via a wp-json/wp/v2/users request.
Base Score:  5.3 MEDIUM

## Exploitation

With an SSH connection to Target 1 I was able to find find the first two flags in the /var/www/ directory with the command

"grep -ri flag ."

This was fairly simple and only required a bit of looking around to discover flag2.txt and after I ran that command above I was able to aquire flag1.txt as well. With flag 1 and 2 found I investigated this directory further and found the wordpress directory. Within this was wp-config.php which wasn't password protected and I had permission on the account I was using to cat the file. Within it was the login info for root for the MySQL database which was root:R@v3nSecurity

commands used: grep -ri flag .
               cat wp-config.php

With this login information I logged into the MySQL database as root and within it found the hashed passwords of the users
steven and michael.

command used: mysql --user=root --password=R@v3nSecurity --host=192.168.1.110

As I was already using michaels account I decided to target the password for steven's account using john, by creating a text file with the password hash for steven in it called password.txt and found after successfully cracking the password the login info is steven:pink84

command used: john password.txt

I then switched users from michael to steven and found that steven had sudo privaleges to execute python commands. With this I used the python command: 
sudo python -c ‘import pty;pty.spawn(“/bin/bash”);’

This command spawns a shell with root privaleges which was used to find the last two flags. One of which was flag4.txt which is found in /root/ and the other was actually in the wordpress MySQL database in the wp_posts section. This was in a slight reverse order and if you managed to find flag 3 first you would find flag 4 with it as well with the same command.

Within this whole explotation section the only real exploit used was the python command to spawn a shell. Besides that everything was found by just investigating a very poorly secured machine. This could have be done in a different way but the way described above relied on the least amount of exploits and just leveraged the poor security configuration of the system to gain root access.

Below is a table of the flags and the commands used to aquire them.

| Flag 	| Commands used                                                              	| Flag string                      	|
|------	|-----------------------------------------------------------------------------|----------------------------------	|
| 1    	| (within /var/www/) grep -ri flag .                                         	| b9bbcb33e11b80be759c4e844862482d 	|
| 2    	| (within /var/wwww/) grep -ri flag .                                        	| fc3fd58dcdad9ab23faca6e9a36e581c 	|
| 3    	| (within MySQL database) use wordpress; show tables; select from wp_posts    | afc01ab56b50591e7dccf93122770cd2 	|
| 4    	| (within /root/) cat flag4.txt	                                              | 715dea6c055b9fe3337544932f2941ce 	|

(Note some flags may be 1 or 2 characters off as I had to type them out by hand)



















