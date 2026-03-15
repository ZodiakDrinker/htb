This is a WebPage under the domain `cctv.htb` .
## Second Nmap
Seems that it uses Apache according to Nmap.
```Shell
➜ nmap -sVC -p22,80 10.129.2.51 --min-rate 5000  
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-07 21:20 +0100
Nmap scan report for cctv.htb (10.129.2.51)
Host is up (0.051s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  256 76:1d:73:98:fa:05:f7:0b:04:c2:3b:c4:7d:e6:db:4a (ECDSA)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: SecureVision CCTV & Security Solutions
Service Info: Host: default; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 73.52 seconds
```
## Page View
The content of the page doesn't looks like to have too much content, asside of the _Staff Login_ button.
![[web_content.png]]
The button redirects to `http://cctv.htb/zm`. It redirects to a login handled by _ZoneMinder_.
![[login_zm.png]]

The credentials for this are User: `admin` Password: `admin`. With this we are able to see that the version of zm is `v1.37.63` .

Found the following exploit about creating monitors on [ExploitDB](https://www.exploit-db.com/exploits/51902)

As that exploit does not work, we will test with a Boolean-Based SQL Injection [CVE-2024-51482](https://github.com/ZoneMinder/zoneminder/security/advisories/GHSA-qm8h-3xvf-m7j3) 
## Boolean-Based SQL Injection
Using a custom python script we extracted the following users that we extracted using `hashcat` 

```
superadmin:$2y$10$cmytVWFRnt1XfqsItsJRVe/ApxWxcIk9cURnm5N.rhlULwM0jrtbm
mark:$2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG. -> opensesame
admin:$2y$10$t5z8uIT.n9uCdHCNidcLf.39T1Ui9nrlCkdXrzJMnJgkTiAvRUM6m
```
## subdirectory listing
using gobuster, I have found an strange file located on `http://cctv.htb/zm/vendor/composer/installed.json`

```JSON
[
    {
        "name": "firebase/php-jwt",
        "version": "v5.0.0",
        "version_normalized": "5.0.0.0",
        "source": {
            "type": "git",
            "url": "https://github.com/firebase/php-jwt.git",
            "reference": "9984a4d3a32ae7673d6971ea00bae9d0a1abba0e"
        },
        "dist": {
            "type": "zip",
            "url": "https://api.github.com/repos/firebase/php-jwt/zipball/9984a4d3a32ae7673d6971ea00bae9d0a1abba0e",
            "reference": "9984a4d3a32ae7673d6971ea00bae9d0a1abba0e",
            "shasum": ""
        },
        "require": {
            "php": ">=5.3.0"
        },
        "require-dev": {
            "phpunit/phpunit": " 4.8.35"
        },
        "time": "2017-06-27T22:17:23+00:00",
        "type": "library",
        "installation-source": "dist",
        "autoload": {
            "psr-4": {
                "Firebase\\JWT\\": "src"
            }
        },
        "notification-url": "https://packagist.org/downloads/",
        "license": [
            "BSD-3-Clause"
        ],
        "authors": [
            {
                "name": "Neuman Vong",
                "email": "neuman+pear@twilio.com",
                "role": "Developer"
            },
            {
                "name": "Anant Narayanan",
                "email": "anant@php.net",
                "role": "Developer"
            }
        ],
        "description": "A simple library to encode and decode JSON Web Tokens (JWT) in PHP. Should conform to the current spec.",
        "homepage": "https://github.com/firebase/php-jwt"
    },
    {
        "name": "ircmaxell/password-compat",
        "version": "v1.0.4",
        "version_normalized": "1.0.4.0",
        "source": {
            "type": "git",
            "url": "https://github.com/ircmaxell/password_compat.git",
            "reference": "5c5cde8822a69545767f7c7f3058cb15ff84614c"
        },
        "dist": {
            "type": "zip",
            "url": "https://api.github.com/repos/ircmaxell/password_compat/zipball/5c5cde8822a69545767f7c7f3058cb15ff84614c",
            "reference": "5c5cde8822a69545767f7c7f3058cb15ff84614c",
            "shasum": ""
        },
        "require-dev": {
            "phpunit/phpunit": "4.*"
        },
        "time": "2014-11-20T16:49:30+00:00",
        "type": "library",
        "installation-source": "dist",
        "autoload": {
            "files": [
                "lib/password.php"
            ]
        },
        "notification-url": "https://packagist.org/downloads/",
        "license": [
            "MIT"
        ],
        "authors": [
            {
                "name": "Anthony Ferrara",
                "email": "ircmaxell@php.net",
                "homepage": "http://blog.ircmaxell.com"
            }
        ],
        "description": "A compatibility library for the proposed simplified password hashing algorithm: https://wiki.php.net/rfc/password_hash",
        "homepage": "https://github.com/ircmaxell/password_compat",
        "keywords": [
            "hashing",
            "password"
        ]
    }
]

```

