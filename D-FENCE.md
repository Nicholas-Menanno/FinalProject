# Blue Team: Summary of Operations

## Network Topology

Insert Image here

## Descriptions of VM's

| VM 	| Name         	| IP Address    	| Description                                  	| OS         	|
|----	|--------------	|---------------	|----------------------------------------------	|------------	|
| 0  	| Hyper-V Host 	| 192.168.1.1   	| Hosts Virtual Network VM's                   	| Windows 10 	|
| 1  	| Kali-Linux   	| 192.168.1.90  	| Attacker Machine                             	| Kali-Linux 	|
| 2  	| Target 1     	| 192.168.1.110 	| Vulnerable machine hosting wordpress website 	| Debian     	|
| 3  	| Target 2     	| 192.168.1.115 	| Optional Vulnerable Machine                  	| Debian     	|
| 4  	| Capstone     	| 192.168.1.105 	| Log Forwarding                               	| Ubuntu     	|
| 5  	| ELK-Server   	| 192.168.1.100 	| Log aggregation SIEM Tool                    	| Ubuntu     	|

###### 192.168.1.100:5601 is for Kibana Data visualization

For the purpose of the report we will only be covering Target 1.
Target 1 has an IP address of 192.168.1.110
It is an apache web server which has ports open;
Port Service

22   SSH
80   HTTP
111  rpcbind
139  Netbios-ssn
445  Netbios-ssn


## Monitoring Targets

To monitor Target 1 there was 3 basic alerts chosen which were alerts for HTTP Errors, HTTP Request size, and CPU Usage.

Alert 1

Excessive HTTP Errors
Using the Packbeat indice use the rules:

WHEN count() GROUPED OVER top 5 'http.response.status\_code' IS ABOVE 400 FOR THE LAST 5 minutes

This alert monitors the amount of error codes being generated, this is useful for monitoring for any issues or problems
that occur with the website being hosted or different types of attacks which would generate error codes as they are performed

Alert 2

HTTP Request Size Monitor
Using the Packetbeat indice use the rules:

WHEN sum() of http.request.bytes OVER all documents IS ABOVE 3500 FOR THE LAST 1 minute

This alert monitors the amount of requests being made to the server which can show either abnormal server traffic being abnormal
or certain types of denial of service attacks like Slowloris attacks which maintain large amounts of open requests to slow down
and eventually crash the server.

Alert 3

CPU Usage Monitor
Using the Metricbeat indice use the rules:

WHEN max() OF system.process.cpu.total.pct OVER all documents IS ABOVE 0.5 FOR THE LAST 5 minutes

This alert will monitor the VM's proccessing and will send an alert if they are working at an abnormal rate, either due to
an influx of server traffic or more likely a Denial of Service attack which wastes the computers proccessing power to crash
the system.




## Patching Security Vulnerabilities


Vulnerability 1, Insecure SSH

To strengthen the security of the SSH service I would recommend a simple program called Fail2Ban. This service ban's IP's after
a certain amount of failed requests are made. Below is a installation guide.

Installing fail2ban

sudo apt-get update
sudo apt-get install fail2ban

to ensure it starts on system startup

sudo systemctl enable fail2ban.service

then use

sudo nano /etc/fail2ban/jail.conf

Within the file configure it to the following settings

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 300
bantime = 3600

now close the file and save it then run

sudo systemctl restart fail2ban.service

and now a basic SSH protection service is configured an running where it will ban IP's from bruteforcing SSH passwords.
A very important step with securing SSH as well would be to have a strong password policy including complex unique passwords
that expire and are not allowed to be reused.

## Vulnerability 2, rpcbind DoS

rpcbind had a flaw which if exploited would cause a denial of service.
A patch was made to prevent DoS attacks from rpcbind with the package from this link

https://launchpad.net/ubuntu/+source/rpcbind/0.2.3-0.6ubuntu0.18.04.2

This is a simple install and will patch that vulnerability

## Vulnerability 3, Wordpress User Enumeration

Wordpress user enumeration allows a malicious actor to gain usernames to attempt bruteforces on passwords for those users.
To stop this install the free WP Hardening plugin, go to the security fixers tab and enable Stop User enumeration
There are various other settings that could be used here to add extra security to the wordpress installation

## Vulnerability 4, wp-config.php is easily accessable

The wp-config.php file is an important file which holds the details about the deployment of wordpress and its configuration.
It is a good idea to change the permissions to read this file to only certain accounts based on the needs
of the developers, which should be heavily restricted as it is not important for most accounts to have access to
and only serves as a liability if it can be accessed easily.
This can be done by creating a group on linux and adding users to it who need to access the wp-config.php file
or other important wordpress files and remove and other accounts from being able to read write or execute any files
in that directory.

## Extra Security: Firewall

As an additional step it would be a good idea to utilize a firewall to have an extra layer of protection for the
system. This can be done with the utility ufw. This can be configured in many different ways but as a general
guideline it would be good to only allow traffic from the open ports and for SSH to deny all traffic unless
a whitelisted IP attempts to access it. There is also an option for enabling logs which can be then
paired with Kibana to monitor the firewall.

