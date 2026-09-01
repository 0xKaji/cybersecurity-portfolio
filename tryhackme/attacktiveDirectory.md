# [Attacktive Directory | TryHackMe](https://tryhackme.com/room/attacktivedirectory)

This report outlines the complete compromise of a Windows Active Directory environment. The engagement began with unauthenticated username enumeration, followed by an AS-REP Roasting attack against accounts lacking Kerberos pre-authentication to secure initial access. Subsequent enumeration of SMB shares uncovered exposed backup credentials. Finally, excessive directory replication privileges assigned to the backup account were exploited via a DCSync attack to dump the domain's NTDS.dit database, enabling full Domain Controller takeover through a Pass-the-Hash attack.
## Reconnaissance

The first step was to enumerate all TCP ports and identify the services running on the target machine, to do this we use **Nmap**.

<img width="557" height="593" alt="Captura de pantalla_2" src="https://github.com/user-attachments/assets/6370201e-8c3a-4aad-850e-aa486a2d6161" />\
<br>

The scan revealed several Windows-related services. Upon further examination, port 3389 (RDP) revealed Windows host and domain information:
  * NetBIOS_Domain_Name: THM-AD
  * DNS_Domain_Name: spookysec.local

These findings confirmed that the target belonged to a Windows Active Directory environment and provided useful domain information for the next enumeration steps.

Windows AD uses Kerberos as an authentication service running on port 88. 
We used **Kerbrute** to perform Kerberos-based username enumeration. By observing the KDC’s responses to authentication requests, we can identify valid domain usernames without authenticating successfully. A user wordlist is provided by the creator for user enumeration. \
<br>

<img width="708" height="437" alt="Captura de pantalla_4" src="https://github.com/user-attachments/assets/ec431e46-f8b3-484e-a6ba-194ebaa93fc6" />\
<br>

We have discovered multiple accounts, notably a service admin account and a backup account.

## Kerberos Exploitation

Using that information we can attempt to exploit Kerberos with an attack method called AS-REP Roasting.

> AS-REP Roasting targets user accounts for which Kerberos pre-authentication is disabled. In this situation, an attacker can request an AS-REP for a valid user without first proving knowledge of the user’s password. The response contains encrypted data derived from the user’s password, which can then be cracked offline.

We will use one of Impacket's tools called GetNPUsers.py, this script allows us to query ASREPRoastable accounts from the Key Distribution Center.

<img width="1373" height="263" alt="Captura de pantalla_5" src="https://github.com/user-attachments/assets/6052b91b-d9a7-4206-a330-337b531e50b4" />\
<br>

Next, we use **Hashcat** to crack the “Kerberos 5 AS-REP etype 23” hash retrieved from the KDC.

<img width="1085" height="164" alt="Captura de pantalla_13" src="https://github.com/user-attachments/assets/b3656a87-c2bc-4b9b-b48d-a4d7bd5e2299" />

## SMB Enumeration

Now we have a set of credentials we can use to enumerate any shares in the domain controller.

<img width="534" height="209" alt="Captura de pantalla_7" src="https://github.com/user-attachments/assets/25f97942-cd4c-442c-ae0f-f782040ae3db" />

<img width="981" height="231" alt="Captura de pantalla_8" src="https://github.com/user-attachments/assets/f1263feb-655f-4236-a30b-8a4ac2039d66" />

<img width="403" height="118" alt="Captura de pantalla_9" src="https://github.com/user-attachments/assets/2d3fb58d-9e6e-4156-8075-c1cbca491959" />\
<br>

We discover a set of encoded backup credentials in the SMB share belonging to the backup user account.

## Domain Privilege Escalation

According to the challenge description, the backup account has a unique permission that allows all Active Directory changes to be synced with this user account, including password hashes.
We can use another tool within **Impacket** called Secretsdump.py to dump all the password hashes the backup account has stored.

The “secretsdump.py” uses the DRSUAPI method to get NTDS.DIT secrets.

> The Ntds. dit file is a database that stores Active Directory data, including information about user objects, groups and group membership. Importantly, the file also stores the password hashes for all users in the domain.

<img width="1078" height="568" alt="Captura de pantalla_10" src="https://github.com/user-attachments/assets/f99fa25e-483c-40ec-8f9d-2ab8649f7575" />\
<br>

We found the administrators NTLM hash that we can use in **Evil-WinRM** to access the target machine using the Pass The Hash attack.

> Pass the Hash attack is a technique whereby an attacker captures a password hash (as opposed to the password characters) and then simply passes it through for authentication and potentially lateral access to other networked systems.
\
<br>

<img width="1036" height="162" alt="Captura de pantalla_11" src="https://github.com/user-attachments/assets/a2a34f2e-21ca-4aa2-995d-2b3b849a3933" />


<img width="471" height="148" alt="Captura de pantalla_12" src="https://github.com/user-attachments/assets/72c58cc6-d716-421b-b52f-13d371accdf0" />

## Conclusion

The attack could have been prevented by enforcing Kerberos pre-authentication on all applicable accounts, using strong and unique passwords, restricting access to SMB shares, protecting backup credentials and auditing directory-replication permissions. Replication privileges should be assigned only to trusted domain controllers and strictly required accounts.


### MITRE ATT&CK Mapping

   Reconnaissance & Discovery:

   * [T1589.002] Gather Victim Identity Information: Usernames (Kerbrute domain enumeration).

   * [T1039] Data from Network Shared Drive (SMB share enumeration for backup credentials).

   Credential Access:
     
   * [T1558.004] Steal or Forge Kerberos Tickets: AS-REP Roasting (Extracting KDC hashes with GetNPUsers).

   * [T1110.002] Password Cracking (Offline cracking of AS-REP tickets via Hashcat).

   * [T1003.006] OS Credential Dumping: DCSync (NTDS.dit extraction using Secretsdump).

   Lateral Movement & Execution:

   * [T1550.002] Use Alternate Authentication Material: Pass the Hash (Authenticating with the Administrator NTLM hash).

   * [T1021.006] Remote Services: Windows Remote Management (Executing remote sessions with Evil-WinRM).
