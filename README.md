# Incident Investigation: NetSupport Manager RAT

## Scenario

While monitoring a SIEM system, multiple alerts were observed indicating potential NetSupport Manager RAT activity from external IP **45.131.214.85** over TCP port 443.

The activity started on **2026-02-28 at 19:55 UTC**.

A packet capture (PCAP) was obtained for further investigation to identify the infected host and associated user within the internal network.

---

## Environment

- LAN Range: 10.2.28.0/24  
- Domain: easyas123.tech  
- AD Name: EASYAS123  
- Domain Controller: 10.2.28.2  
- Gateway: 10.2.28.1  

---

## Objective

The goal of this investigation is to analyze the PCAP file and identify:

- The infected Windows client IP address  
- The MAC address of the infected system  
- The hostname of the infected system  
- The user account involved  
- The full name of the user  

---

## Dataset Source

The PCAP used in this investigation was obtained from:

https://www.malware-traffic-analysis.net/

This dataset is publicly available and used for malware traffic analysis and security training.

---

## Tools Used

- Wireshark (primary tool for packet analysis)
- Manual traffic analysis using filters and protocol inspection

---
## Investigation Process

## Investigation Process

### Step 1: Identify suspicious traffic

To investigate the alerts, traffic was filtered for communication with the suspicious external IP:

```wireshark
ip.addr == "45.131.214.85"
```

This revealed multiple connections over TCP port 443, indicating potential encrypted command-and-control (C2) communication.

---

### Step 2: Identify infected internal host

By reviewing the filtered traffic, the internal IP communicating with the malicious server was identified as:


```wireshark
Filter : ip.scr == 45.131.214.85
ip.addr == "10.2.28.88"
```

This system was consistently initiating connections to the external IP, and only one internal ip communicate with suspicous external ip

---

### Step 3: Extract MAC address

After identifying the infected IP, Ethernet frame details were inspected to determine the MAC address.

The associated MAC address was:
```wireshark
Filter : ip.src == 45.131.214.85 (Showed in Ethernet II,src)
Mac Address infected system: "00:19:d1:b2:4d:ad"
```
---

### Step 4: Identify hostname

To determine the hostname of the infected system, DNS and NBNS traffic were analyzed. so i started with nbns first and.

The hostname was identified as:

```wireshark
Filter : nbns and ip.addr == 10.2.28.88
Host Name:Name: "DESKTOP-TEYQ2NR"
```

---

### Step 5: Identify user account

Initial analysis focused on authentication-related traffic such as SMB/NTLM; however, no relevant user information was observed in those protocols. Further inspection of DNS traffic revealed a CNAME string that contained the username associated with the infected system.

The user account identified was:
```wireshark
Filter: kerberos.CNameString and ip.addr == 10.2.28.88
user account name : "brolf"
```
---

### Step 6: Identify full name of user

Identifying the full name of the user was challenging. I initially analyzed SMB, HTTP, and Kerberos traffic but did not find relevant information. I then performed a keyword-based search for "Full Name" within the packet capture ultimately revealed the full name associated with the user account.

```wireshark
Use Search options:brolf
Full Name: "Becka Rolf"

```

- **Infected IP Address: "10.2.28.88"**  
- **MAC Address: "00:19:d1:b2:4d:ad"**  
- **Hostname:"DESKTOP-TEYQ2NR"**  
- **User Account: "brolf"**  
- **Full Name: "Becka Rolf"**  
## Conclusion

The investigation confirmed that the internal host 10.2.28.88 (DESKTOP-TEYQ2NR) was communicating with a suspicious external IP associated with NetSupport Manager RAT activity.

The presence of this communication indicates a likely compromise of the system and potential unauthorized remote access.

### Recommended Actions:
- Isolate the infected host from the network  
- Reset credentials for user account brolf  
- Perform malware analysis and removal  
- Review additional network activity for lateral movement  
