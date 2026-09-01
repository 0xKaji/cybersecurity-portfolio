# [Smol | TryHackMe](https://tryhackme.com/room/smol)

This security assessment details the full compromise of a Linux server hosting a WordPress application. Initial access was achieved by exploiting a Local File Inclusion (LFI) vulnerability in an outdated plugin, which exposed credentials and revealed a hidden backdoor allowing Remote Code Execution (RCE). Privilege escalation to root was accomplished by cracking database and ZIP file passwords, recovering exposed SSH keys, and abusing misconfigured su and sudo permissions. The attack resulted in complete system and data takeover.

## Reconnaissance

Firstly, we execute **Nmap** to discover services running on the target machine.
We find a ssh service and an Apache http server that redirects us to www.smol.thm. For ease, we add that url to /etc/hosts. 

<img width="817" height="509" alt="Captura de pantalla_2" src="https://github.com/user-attachments/assets/fcfd8489-b277-44c4-85ac-b7a50ed7cc55" />\
<br>

**WhatWeb** reports that the website is running a WordPress service. That WordPress version doesn't seem to be vulnerable, but there may be vulnerable plugins installed.
<img width="1256" height="79" alt="Captura de pantalla_3" src="https://github.com/user-attachments/assets/b3a324ec-d8f7-4f76-8e08-b3e864d7955d" />

<img width="1263" height="93" alt="Captura de pantalla_4" src="https://github.com/user-attachments/assets/a049021a-6697-4028-aa1d-a1885cbf1792" />

The scan shows that the plugin jsmol2wp is installed. This plugin is known for having a Local File Inclusion vulnerability.

> WordPress JSmol2WP plugin 1.07 is susceptible to local file inclusion via ../ directory traversal in query=php://filter/resource= in the jsmol.php query string. An attacker can possibly obtain sensitive information, modify data, and/or execute unauthorized administrative operations in the context of the affected site. This can also be exploited for server-side request forgery. [PoC](https://github.com/sullo/advisory-archives/blob/master/wordpress-jsmol2wp-CVE-2018-20463-CVE-2018-20462.txt)
<br>

Inside wp_config.php we find a WordPress user's credentials. With them, we can access the admin panel for WP.

<img width="577" height="136" alt="Captura de pantalla_5" src="https://github.com/user-attachments/assets/24cd0f56-767d-441f-9df3-791edae54739" />

<img width="1086" height="760" alt="Captura de pantalla_6" src="https://github.com/user-attachments/assets/bedeaa6f-06c2-4a64-aa97-a43d44d8c6c6" />

## Initial access

Sortly after, I found out the user didn't have any meaningful permissions, so i couldn't execute some powerful [scripts](https://github.com/nowak0x01/WPXStrike)

<img width="1149" height="302" alt="Captura de pantalla_7" src="https://github.com/user-attachments/assets/8b8431be-40b2-4041-88ef-8ef7b47028fe" />\
<br>

But, looking through the dashboard we can find a private page with some interesting tasks.

<img width="1149" height="447" alt="Captura de pantalla_8" src="https://github.com/user-attachments/assets/8450e416-65b5-4675-a88d-8d2e067f7e0f" />

<img width="806" height="235" alt="Captura de pantalla_9" src="https://github.com/user-attachments/assets/244f5d2c-dbf7-4e61-8859-7a9c3c4d5fb4" />\
<br>

Now we know that the plugin Hello Dolly is installed alongside jsmol2wp. We can use the previous found LFI to examine this new plugins source code.

<img width="1280" height="46" alt="Captura de pantalla_10" src="https://github.com/user-attachments/assets/4bf97fa5-09ea-4982-a98b-c27913bf0e9c" />
<img width="1037" height="40" alt="Captura de pantalla_11" src="https://github.com/user-attachments/assets/342ad6b2-3a73-4ee7-8ced-5228ae3c221b" />
<img width="929" height="131" alt="Captura de pantalla_12" src="https://github.com/user-attachments/assets/563b44ce-9453-48c1-947b-6eb16bee54a3" />

The codified sequence represents the word cmd
    \143 = c
    \155 = m
    \x64 = d

## WordPress Exploitation

This seems like a possible RCE (Remote Code Execution). Testing on different places of the site i finally found out that using the cmd parameter in index.php executed the code.

<img width="549" height="106" alt="Captura de pantalla_14" src="https://github.com/user-attachments/assets/fe2c5c99-2c00-4bb8-bb3e-b9e99a0f21fa" />

So we reverse shell.

<img width="545" height="92" alt="Captura de pantalla_15" src="https://github.com/user-attachments/assets/f2646521-7887-44b7-81ec-7ba49b12534e" />

Since we are inside the WordPress server we can find inside it's database all the users and hashed passwords with the credentials we already have:
`mysql -u wpuser --password=kbLSF2Vop#lw3rjDZ629*Z%G -h localhost -e "use wordpress;select concat_ws(':', user_login, user_pass) from wp_users;"`

<img width="368" height="160" alt="Captura de pantalla_16" src="https://github.com/user-attachments/assets/310d47a8-55e8-44df-81cc-74c52c5190fd" />

One of this passwords can be cracked using **JohnTheRipper** and rockyou. 

<img width="807" height="308" alt="Captura de pantalla_18" src="https://github.com/user-attachments/assets/c1c24628-f6fc-46df-8a9f-4b4e5acca349" />\
<br>

This password belongs to the user diego. Inside his home directory we can find the user flag.

<img width="597" height="161" alt="Captura de pantalla_17" src="https://github.com/user-attachments/assets/3c6c725b-ce29-4c5a-b51f-8ff89f2ae71a" />

## Privilege Escalation

After some digging, i found a ssh key belonging to the user think that was readable.

<img width="572" height="145" alt="Captura de pantalla_19" src="https://github.com/user-attachments/assets/24fe67e1-49cd-4129-b190-f481b1d2d465" />

Now, using **LinPeas** we find that su is configured so that the user think can log into user gege without password.

<img width="579" height="177" alt="Captura de pantalla_20" src="https://github.com/user-attachments/assets/bcfd3dfb-18f8-4f65-940d-38638d967b64" />

Inside gege's home directory there is a zip file that looks promising, a WordPress backup. Since it is protected by a password, we will need to once again crack it using **JTR**. So i download it in my host machine.

<img width="1229" height="219" alt="Captura de pantalla_22" src="https://github.com/user-attachments/assets/34100f94-5ec1-42c7-965c-b91aba56bed6" />
<img width="261" height="44" alt="Captura de pantalla_23" src="https://github.com/user-attachments/assets/3d03a806-540d-49a1-8fa2-7d4c2aec515e" />

<img width="732" height="172" alt="Captura de pantalla_24" src="https://github.com/user-attachments/assets/c0a67785-3d2c-427b-be59-15dabb975440" />\
<br>

Once we find the password, we unzip the file. Inside wp-config, we can find once again some user credentials. This time belonging to user xavi.

<img width="662" height="496" alt="Captura de pantalla_25" src="https://github.com/user-attachments/assets/1b6531c7-67ba-4130-a736-0fb2e84ce578" />

User xavi is capable of executing any sudo command, so we just sudo su into root.

<img width="947" height="112" alt="Captura de pantalla_26" src="https://github.com/user-attachments/assets/9d3d581e-9fe1-4b17-a5c2-71c6c01f9d1f" />


<img width="950" height="150" alt="Captura de pantalla_27" src="https://github.com/user-attachments/assets/3183e588-e0fb-498e-860b-ac43bddd4469" />

## Conclusion

This attack could have been prevented by keeping WordPress and all of its plugins updated, removing unnecessary components, and applying strict input validation to prevent arbitrary code execution. Sensitive information, such as database credentials, SSH keys, configuration files, and backups, should also have been securely stored and protected with appropriate permissions. In addition, applying the principle of least privilege would have prevented unnecessary access between users and restricted unrestricted sudo permissions. Finally, regular vulnerability assessments, password audits, and system monitoring would have helped identify and fix these security weaknesses before they could be exploited.

### MITRE ATT&CK Mapping

Initial Access: [T1190] Exploit Public-Facing Application (jsmol2wp plugin LFI).


Execution:

* [T1059.004] Command and Scripting Interpreter: Unix Shell (Reverse shell).

*  [T1505.003] Server Software Component: Web Shell (Hidden 'cmd' parameter in index.php).

Credential Access:

* [T1552.001] Credentials In Files (Reading wp-config.php).

* [T1552.004] Private Keys (Recovering user 'think' SSH key).

* [T1110.002] Password Cracking (JohnTheRipper for DB and ZIP hashes).

Privilege Escalation & Lateral Movement:

* [T1078.003] Valid Accounts: Local Accounts (Pivoting between users).

* [T1548.003] Abuse Elevation Control Mechanism: Sudo and Sudo Caching (Exploiting unrestricted sudo rights).




