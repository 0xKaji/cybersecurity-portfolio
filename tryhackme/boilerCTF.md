# [Boiler CTF | TryHackMe](https://tryhackme.com/room/boilerctf2)

## Recon

The first step is to identify which TCP ports are open on the target machine. An **Nmap** scan was performed against the host to scan all TCP ports and identify the services and versions running on them.****

<img width="688" height="778" alt="Captura de pantalla_10" src="https://github.com/user-attachments/assets/b2ad4810-be80-4910-8d83-5e235b91a864" />

The FTP service allows anonymous access, as indicated by the scan results. 
> Anonymous FTP access allows users to connect to an FTP server without providing valid credentials. Although it is sometimes intentional, it can expose sensitive files or provide useful information about the target.

<img width="536" height="113" alt="Captura de pantalla_11" src="https://github.com/user-attachments/assets/ad8945c9-24ab-443d-9a62-2b128fc888dd" />

<img width="591" height="60" alt="Captura de pantalla_12" src="https://github.com/user-attachments/assets/3643e71f-c980-4545-a699-c0714acad6e2" />

The contents of the file appear to be encoded or obfuscated rather than written in plain English. The text follows a simple substitution pattern consistent with ROT13.
> ROT13 is a simple substitution cipher that replaces each letter with the letter located 13 positions later in the alphabet. Applying ROT13 twice restores the original text.

<img width="1030" height="362" alt="Captura de pantalla_13" src="https://github.com/user-attachments/assets/bfa39aaa-0c73-4ec8-b13a-96f9d7c998c1" />

The message indicates that we should continue enumerating the target. We can further investigate the Apache HTTP server by enumerating its directories with **Gobuster**.

<img width="675" height="462" alt="Captura de pantalla_14" src="https://github.com/user-attachments/assets/dc109e2c-a9a6-4c2a-92ec-6b1e9cd8b3a4" />

The discovery of the /joomla directory provides the main application path for further assessment.

<img width="858" height="519" alt="Captura de pantalla_16" src="https://github.com/user-attachments/assets/cfd3d95f-72c8-42b3-ac3e-13e0b31ced56" />

As we dont have any credentials we can't access the administration panel. We will need to find them or find another route.

<img width="861" height="705" alt="Captura de pantalla_17" src="https://github.com/user-attachments/assets/93ac94c3-0f70-4249-8071-518091d4dd3d" />

This Joomla version does not appear to be vulnerable. Since we do not have valid credentials, we cannot access the administration panel. We will need to find them or identify another way to gain access.

<img width="786" height="745" alt="Captura de pantalla_18" src="https://github.com/user-attachments/assets/45d1e46c-6084-4a1d-820a-6ab84c396704" />

There are multiple interesting directories, but the one we should focus on is _test.
Inside it, we find sar2html.

## Exploitation

<img width="993" height="572" alt="Captura de pantalla_19" src="https://github.com/user-attachments/assets/ff4126de-c6c1-43ec-8f50-a27674a1decd" />

A quick search reveals a [remote code execution (RCE) vulnerability](https://www.exploit-db.com/exploits/47204) affecting this application. We can test whether it is exploitable on the target.

> Remote code execution (RCE) is a vulnerability that allows an attacker to execute arbitrary commands on a remote system. Depending on the privileges of the vulnerable service, this can lead to unauthorized access or privilege escalation.

<img width="497" height="275" alt="Captura de pantalla_20" src="https://github.com/user-attachments/assets/5f20506f-189e-4d91-a166-2df46d3a1556" />

The exploit works, and we obtain a reverse shell.

<img width="538" height="138" alt="Captura de pantalla_21" src="https://github.com/user-attachments/assets/17b2de6c-5852-4d6d-9bba-2ef32751ba31" />

<img width="939" height="175" alt="Captura de pantalla_22" src="https://github.com/user-attachments/assets/3f3b8408-e0b8-48fb-b319-78dd56ca64c4" />

log.txt contains basterd’s password in plain text. We can use these credentials to connect to the machine via SSH.

## Privilege escalation

<img width="669" height="721" alt="Captura de pantalla_24" src="https://github.com/user-attachments/assets/a48b9706-49f0-4d24-84da-2aa857581767" />

We find another plain-text password inside a backup script, this time belonging to stoner. Since basterd does not have sudo privileges and is not a member of any particularly useful group, we switch to the stoner account.
<img width="435" height="95" alt="Captura de pantalla_25" src="https://github.com/user-attachments/assets/347a3f6a-5648-472e-808e-59261cb5e9db" />

We find the first flag in this account’s home directory.

<img width="1033" height="33" alt="Captura de pantalla_27" src="https://github.com/user-attachments/assets/7addcaec-195d-4f1a-927f-07426c25b23c" />

The stoner user belongs to the lxd group. Membership in this group can potentially be abused for [privilege escalation vulnerability](https://github.com/initstring/lxd_root). However, we continue looking for a simpler method.

> LXD is a container manager for Linux. Users who belong to the lxd group may be able to create or manage privileged containers, which can sometimes be abused to access the host system as root.

<img width="1214" height="447" alt="Captura de pantalla_28" src="https://github.com/user-attachments/assets/bd6c3040-f37b-4231-ab33-2ccd72b994ba" />

Using **LinPeas** we discover that the find binary has the SUID permission set.

<img width="848" height="498" alt="Captura de pantalla_30" src="https://github.com/user-attachments/assets/47950fa5-aa9e-4ec6-9302-5c3104de6a84" />

Fortunately, for us [GTFOBins](https://gtfobins.org/) provides a straightforward way to exploit this misconfiguration and obtain a root shell.

<img width="523" height="147" alt="Captura de pantalla_29" src="https://github.com/user-attachments/assets/8fcc95ac-7129-494f-9a05-022f851659b9" />

