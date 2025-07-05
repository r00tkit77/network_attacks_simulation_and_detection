**DISCLAIMER:** This project is for ***educational purposes only***. All attacks were performed in a controlled lab environment on devices I own. Never use these techniques on unauthorized networks or systems.
<br><br><br>

**Lab Setup:**
<br><br>
Attack OS: Parrot Security 6.4

Target WLAN: 192.168.1.0/24

Attacker IP: 192.168.1.16

Gateway IP: 192.168.1.1

Tools Used: Nmap, Hydra, Ping, Hping3, Arpspoof, TCPDump, Wireshark
<br><br><br>

**Traffic Capture:**
<br><br>
Using TCPDump on the attack host to capture all traffic on my WLAN interface.

*tcpdump -i wlp0s20f3*

![image](https://github.com/user-attachments/assets/4be930a3-8b82-422f-8da5-b41fe11b4b53)
<br><br><br>

**Attacks:**
<br><br>

Here begins the fun part. First I will begin with **ping sweep** using nmap to find active hosts in the subnet.

*nmap -sn 192.168.1.0/24*

![image](https://github.com/user-attachments/assets/d3f2c3de-52e1-4f4f-9447-5c03f547b072)

Now a *TCP syn scan* and *UDP scan* respectively with OS detection on the identified hosts with nmap.

*nmap -sS -O 192.168.1.17*
 
![image](https://github.com/user-attachments/assets/e725e899-692a-465f-98a0-9fd0b5edfd77)

*nmap -sU -O 192.168.1.3*
 
![image](https://github.com/user-attachments/assets/54012349-b8c7-4a3c-9184-d894d3a39f20)
<br><br><br>
Now lets move to *brute force* attack. Using hydra to attack open ftp port 2221 in one of the hosts.

*hydra -l admin -P /usr/share/wordlists/rockyou.txt ftp://192.168.1.9:2221*

![image](https://github.com/user-attachments/assets/dd7b0a26-0cc4-4653-9201-605c03cf133c)
<br><br>

Time for some DOS. *ICMP Flood* attack using linux's ping utility.

*ping -f 192.168.1.9*

![image](https://github.com/user-attachments/assets/5cc06116-6093-412a-bcae-08af3eb6f5aa)
<br><br>

Another DOS, this time *TCP syn Flood* using hping3 on port 2221.

*hping3 -S --flood -p 2221 192.168.1.9*

![image](https://github.com/user-attachments/assets/dc27073b-3e2b-495c-a8e0-d94c0e7ca660)
<br><br>

The final attack: *Man in the Middle Attack with ARP Spoofing* to intercept traffic between a host and the gateway.

*arpspoof -i wlp0s20f3 -t 192.168.1.9 192.168.1.1*

![image](https://github.com/user-attachments/assets/0d059ee7-09e7-4e7a-9f81-d83469ca2042)

*arpspoof -i wlp0s20f3 -t 192.168.1.1 192.168.1.9*

![image](https://github.com/user-attachments/assets/f75e9d47-39d5-42dc-b2bd-16edf78ea902)
<br><br><br>

**Analysing Captured Traffic with Wireshark:**
<br><br>

Opening the .pcapng file in Wireshark.

![image](https://github.com/user-attachments/assets/0092e8a9-e2ce-448d-b08b-f336d12f6d46)
<br><br>
Statistics > Capture file properties:

![image](https://github.com/user-attachments/assets/c14c337d-2f5e-4a44-a733-5d099bda5419)
<br><br>
Statistics > Endpoints:

![image](https://github.com/user-attachments/assets/3e505fb3-bab0-489f-a360-d5419f5233c2)
<br><br>
Statistics > Conversations:

![image](https://github.com/user-attachments/assets/092ba8fb-7ebe-412d-b4a0-2f5b6e6a1999)
<br><br>

Now we will use wireshark filters to detect the attacks conducted.
<br><br>

To detect ping sweep, using filter *arp* as nmap by default uses ARP requests on a LAN for ping scan.

![image](https://github.com/user-attachments/assets/b9290113-14fc-4207-8fb7-b98627f59a6a)

ARP broadcast in the subnet asking for every host's MAC address confirms ping sweep. However, it can also be ARP request flood attack but here we have one request per host in sequential order.
<br><br>

For detecting nmap TCP scans, use the filter *tcp.flags.syn==1 and tcp.flags.ack==0*. Nmap TCP Connect scans complete the 3 Way TCP Handshake while the Syn scan doesn't. However, in both the cases the first step of the 3 way handshake is done which involves sending a TCP packet with only the syn flag set to 1. 

![image](https://github.com/user-attachments/assets/a2e92b9f-2c8b-40bf-bb80-efb05f71539c)

The filtered packets show a host sending TCP syn packets to various commonly used ports, confirming the TCP scan.
<br><br>

**Note:** To differentiate between TCP connect and syn scans, follow TCP stream of a suspected packet to check if the handshake was completed or not.

![image](https://github.com/user-attachments/assets/8e3dbc05-8081-47fe-8c39-87ccc73d6633)
A completed handshake confirms a TCP connect scan.

![image](https://github.com/user-attachments/assets/98d82fa2-fa4e-4bc0-9b9c-cde66d78b05d)
An incomplete handshake confims a TCP syn scan.
<br><br>

To detect nmap UDP scans, the filter *(icmp.type==3 and icmp.code==3)* can be used. An open port doesn't reply to UDP scans but a closed port will send ICMP Destination and Port Unreachable error.

![image](https://github.com/user-attachments/assets/c55dc7e6-c5cc-4cb2-91df-c5e8b58b097b)

Multiple ICMP port and destination unreachable errors confirm the UDP scan.
<br><br>

Brute force attacks on FTP can be detected using *ftp.response.code == 530*. FTP error code 530 represents login authentication failure.

![image](https://github.com/user-attachments/assets/021409ba-3ccb-45cc-96e4-02f74cb834c0)

Multiple login failures reflect brute force attack.


To see the attempted usernames and passwords use: *ftp.request.command==”USER”* and *ftp.request.command==”PASS”*

![image](https://github.com/user-attachments/assets/4cc17c73-39b3-4988-a244-bd1b096a311f)

![image](https://github.com/user-attachments/assets/1f2ad56f-67b8-4457-8323-54e5ef84d859)

Spike in I/O Graph also indicates a brute force attack.

![image](https://github.com/user-attachments/assets/9fb4d68e-2291-4ec8-9a75-047c5bc0e7c0)
<br><br>

ICMP floods can be detected with *icmp* to check for huge no. of ICMP requests from a single source.

![image](https://github.com/user-attachments/assets/0edee4ef-b76b-485b-adf6-652d45627546)

The I/O graph indicates the same.

![image](https://github.com/user-attachments/assets/08bafc1d-d949-466c-8ba4-c561e9d558e2)
<br><br>

TCP syn flood can also be detected using *tcp.flags.syn == 1 && tcp.flags.ack == 0*.

![image](https://github.com/user-attachments/assets/d16a4ed7-2870-46c1-8206-cd95f1b6d3b3)

Huge no. of TCP syn packets to the same port of the target indicates TCP syn flooding.

![image](https://github.com/user-attachments/assets/e9ca0b1d-2be6-4968-b095-be4f0ceaa16f)

The I/O graph indicates the same.
<br><br>

To detect the final attack that is ARP Spoofing, we will again use *arp*

![image](https://github.com/user-attachments/assets/350b8774-7604-4e37-a495-8cb8d3d93a16)

Two different IPs (192.168.1.1 and 192.168.19) claiming to have the same MAC address with unsolicited ARP response, confirms a Man in the Middle attack with ARP Spoofing.

![image](https://github.com/user-attachments/assets/2793e2d4-cc5c-4e68-94d5-16db5a9465f0)

The MAC address in the ARP response belongs to the attacker (192.168.1.16).
<br><br><br><br>

That's it for this project. We have successfully conducted various network attacks and then detected them with wireshark.
<br><br>


***THE END***















