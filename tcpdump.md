# TCPDUMP Tutorial

## Basic commands
1. **Everything on an interface**
```bash
tcpdump -i eth0
```

2. **Find traffic by IP**
One of the most common queries, using `host`, you can see traffic that's going to or from 1.1.1.1
```bash
tcpdump host 1.1.1.1
```

3. **Filtering by source and/or destination**
If you only want to see traffic in one direction or the other, you can use `src` and `dst`
```bash
tcpdump src 1.1.1.1
tcpdump dst 1.0.0.1
```

4. **Finding packets by network**
To find packets going to or from a particular network or subnet, use the `net` option
```bash
tcpdump net 1.2.3.0/24
```

5. **Get packet contents with hex output**
Hex output is useful when you want to see the content of the packets in question, and it's often best used when you're isolating a few candidates
```bash
tcpdump -c 1 -X icmp
```

6. **Show traffic related to a specific port**
```bash
tcpdump port 3389
tcpdump src port 1025
```

7. **Show traffic of one protocol**
```bash
tcpdump icmp
```

8. **Show only IP6 traffic**
```bash
tcpdump ip6
```

9. **Find traffic using port ranges**
```bash
tcpdump portrange 21-23
```

10. **Find traffic based on packet size**
```bash
tcpdump less 32
tcpdump greater 64
tcpdump <= 128
```

11. **Reading / Writing captures to a file (pcap)**
```bash
tcpdump port 80 -w capture_file.pcap
tcpdump -r capture_file.pcap
```

## Advanced

### More options
* `-X`: Show the packet's contents in both HEX and ASCII
* `-XX`: Same as `-X`, but also shows the ethernet header
* `-D`: Show the list of available interfaces
* `-l`: Line-readable output
* `-q`: Be less verbose (more quiet) with your output
* `-t`: Give human-readable timestamp output
* `-ttt`: Give maximally human-readable timestamp output
* `-i eth0`: Listen on the eth0 interface
* `-vv`: Verbose output (more v's gives more output)
* `-c`: Only get x number of packets and then stop
* `-s`: Define the snaplength(size) of the capture in bytes. Use `-s0` to get everything, unless you're intentionally capturing less.
* `-S`: Print absolute sequence numbers
* `-e`: Get the ethernet header as well
* `-q`: Show less protocol information
* `-E`: Decrypt IPSEC traffic by providing an encryption key

### It's all about the combinations
#### Combinations
1. AND
  `and` or `&&`
2. OR
  `or` or `||`
3. EXCEPT
  `not` or `!`

#### option combinations
Use this combination to see verbose output, with no resolution of hostnames or port numbers, using absolute sequence numbers, and showing human-readable timestamps.
```bash
tcpdump -ttnnvvS
```

1. **From specific IP and destined for a specific port**
```bash
tcpdump -nnvvS src 10.5.2.3 and dst port 3389
```

2. **From One network to another**
Look for all traffic coming from 192.168.x.x and going to the 10.x or 172.16.x.x networks, and we're showing hex output with no hostname resolution and one level of extra verbosity.
```bash
tcpdump -nvX src net 192.168.0.0/16 and dst net 10.0.0.0/8 or 172.16.0.0/16
```

3. **Non ICMP traffic going to a specific IP**
```bash
tcpdump dst 192.168.0.2 and src net and not icmp
```

4. **Traffic from a host that isn't on a specific port**
```bash
tcpdump -vv src mars and not dst port 22
tcpdump -vv src 192.168.1.199 and not dst port 22 and not src port 22
```

5. **Single quotes**
Keep in mind that when you’re building complex queries you might have to group your options using single quotes. Single quotes are used in order to tell tcpdump to ignore certain special characters—in this case below the “( )” brackets. This same technique can be used to group using other expressions such as host, port, net, etc.
```bash
tcpdump 'src 10.0.2.4 and (dst port 3389 or 22)'
```

6. **Isolate TCP flags**
You can also use filters to isolate packets with specific TCP flags set.
```bash
# Isolate TCP RST flags
tcpdump 'tcp[13]&4!=0'
tcpdump 'tcp[tcpflags]==tcp-rst'

# isolate TCP SYN flags
tcpdump 'tcp[13] & 2!=0'
tcpdump 'tcp[tcpflags]==tcp-syn'

# Isolate packets that have both SYN and ACK flags set
tcpdump 'tcp[13]=18'

# Isolate TCP URG flags
tcpdump 'tcp[13] & 31!=0'
tcpdump 'tcp[tcpflags]==tcp-urg'

# Isolate TCP ACK flags
tcpdump 'tcp[13] & 16 !=0'
tcpdump 'tcp[tcflags] == tcp-ack'

# Isolate TCP PSH flags
tcpdump 'tcp[13] & 8!=0'
tcpdump 'tcp[tcpflags] == tcp-push'

# Isolate TCP FIN flags
tcpdump 'tcp[13] & 1!=0'
tcpdump 'tcp[tcpflags] == tcp-fin'

# Both Syn and RST set
tcpdump 'tcp[13] = 6'

# Find Http User agents
tcpdump -vvAls0 | grep 'User-Agent:'

# Cleartext Get requests
tcpdump -vvAls0 | grep 'GET'

# Find http host headers
tcpdump -vvAls0 | grep "Host:"

# Find Http Cookies
tcpdump -vvAls0 | grep 'Set-Cookie|Host:|Cookie:'

# Find ssh connections
# This one works regardless of what port the connection
# comes in on, because it’s getting the banner response.
tcpdump 'tcp[(tcp[12]>>2):4] = 0x5353482D'

# Find DNS Traffic
tcpdump -vvAs0 port 53

# Find FTP traffic
tcpdump -vvAs0 port ftp or ftp-data

# Find NTP Traffic
tcpdump -vvAs0 port 123

# Find Cleartext Passwords
tcpdump port http or port ftp or port smtp or port imap or port pop3 or port telnet -lA | egrep -i -B5 'pass=|pwd=|log=|login=|user=|username=|pw=|passw=|passwd= |password=|pass:|user:|username:|password:|login:|pass |user '

# Find Traffic With Evil Bit
# There’s a bit in the IP header that never gets set by legitimate applications, which we call the “Evil Bit”. Here’s a fun filter to find packets where it’s been toggled.
tcpdump 'ip[6] & 128 != 0'

```
