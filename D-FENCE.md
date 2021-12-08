﻿Blue team

Network Topology

Insert Image here

Descriptions of VM's

Host VM

Hyper-V Host

192.168.1.1

Hosts Virtual Network VM's

Windows 10 OS

VM 1

Kali Linux

192.168.1.90

Attacker Machine

Kali Linux OS

VM 2

Target 1

192.168.1.110

Vulnerable machine hosting wordpress website

Debian Linux OS

VM 3

Target 2

192.168.1.115

Optional Vulnerable Machine

Debian Linux OS

VM 4

Capstone

192.168.1.105

Alert Testing

Ubuntu Linux OS

VM 5

ELK Server

192.168.1.100

Log aggregation SIEM Tool

Ubuntu Linux OS

192.168.1.100:5601 is for Kibana Data visualization

For the purpose of the report we will only be covering Target 1.

Target 1 has an IP address of 192.168.1.110

It is an apache web server which has ports open;

Port Service

22   SSH

80   HTTP

111  rpcbind

139  Netbios-ssn

445  Netbios-ssn


Monitoring Targets

To monitor Target 1 there was 3 basic alerts chosen which were alerts for HTTP Errors, HTTP Request size, and CPU Usage.

Alert 1

Excessive HTTP Errors

Packbeat

WHEN count() GROUPED OVER top 5 'http.response.status\_code' IS ABOVE 400 FOR THE LAST 5 minutes

This alert monitors the amount of error codes being generated, this is useful for monitoring for any issues or problems

that occur with the website being hosted or different types of attacks which would generate error codes as they are performed

Alert 2

HTTP Request Size Monitor

Packetbeat

WHEN sum() of http.request.bytes OVER all documents IS ABOVE 3500 FOR THE LAST 1 minute

This alert monitors the amount of requests being made to the server which can show either abnormal server traffic being abnormal

or certain types of denial of service attacks like Slowloris attacks which maintain large amounts of open requests to slow down

and eventually crash the server.

Alert 3

CPU Usage Monitor

Metricbeat

WHEN max() OF system.process.cpu.total.pct OVER all documents IS ABOVE 0.5 FOR THE LAST 5 minutes

This alert will monitor the VM's proccessing and will send an alert if they are working at an abnormal rate, either due to

an influx of server traffic or more likely a Denial of Service attack which wastes the computers proccessing power to crash

the system.




Patching Security Vulnerabilities

Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. For each vulnerability/exploit identified by the alerts above, suggest a patch. E.g., implementing a blocklist is an effective tactic against brute-force attacks. It is not necessary to explain how to implement each patch.

Vulnerability 1


Patch: TODO: E.g., install special-security-package with apt-get


Why It Works: TODO: E.g., special-security-package scans the system for viruses every day

Todo: SSH configuration with fail2ban and private keys

Samba security patches

Rpcbind security patch to prevent DoS

Wordpress anti user enumeration

Anti Nmap measures to not show ports

Vulnerability 1, Insecure SSH

Fail2ban patch

Install fail2ban

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

and now a basic SSH protection service is configured an running where it will ban IP's from bruteforcing SSH passwords

Vulnerability 2, rpcbind DoS

A patch was made to prevent DoS attacks from rpcbind with the package from this link

https://launchpad.net/ubuntu/+source/rpcbind/0.2.3-0.6ubuntu0.18.04.2

This is a simple install and will patch that vulnerability

Vulnerability 3, Wordpress User Enumeration

To stop this install the free WP Hardening plugin, go to the security fixers tab and enable Stop User enumeration

There are various other settings that could be used here to add extra security to the wordpress installation

Vulnerability 4, wp-config.php is easily accessable

wp-config.php

I would also recommened to change the permissions to read this file to only certain accounts based on the needs

of the developers, which should be heavily restricted as it is not important for most accounts to have access to

and only serves as a liability if it can be accessed easily.

This can be done by creating a group on linux and adding users to it who need to access the wp-config.php file

or other important wordpress files and remove and other accounts from being able to read write or execute any files

in that directory.

As an additional step it would be a good idea to utilize a firewall to have an extra layer of protection for the

system. This can be done with the utility ufw. This can be configured in many different ways but as a general

guideline it would be good to only allow traffic from the open ports and for SSH to deny all traffic unless

a whitelisted IP attempts to access it. There is also an option for enabling logging which can be then

paired with Kibana to monitor the firewall.










































































