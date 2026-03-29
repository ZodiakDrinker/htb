We are pointing to port 443 as the port 80 has a redirect that goes through the HTTPS server.

As we saw, the webserver has a FQDN which is `kobold.htb`

### Subdomain discovering
```Shell
ffuf -u 'https://10.129.8.34' -H 'Host: FUZZ.kobold.htb' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -c -fs 154

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://10.129.8.34
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.kobold.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

bin                     [Status: 200, Size: 24402, Words: 1218, Lines: 386, Duration: 60ms]
mcp                     [Status: 200, Size: 466, Words: 57, Lines: 15, Duration: 62ms]
:: Progress: [20000/20000] :: Job [1/1] :: 749 req/sec :: Duration: [0:00:30] :: Errors: 0 ::
```

As we can see, there  are 2 new doimains `bin.kobold.htb` and `mcp.kobold.htb`.
