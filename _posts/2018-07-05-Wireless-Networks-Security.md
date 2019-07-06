---
title: Wireless Networks Security
date: 2018-07-05 16:53:38
tags: 
    - GSM
    - 3G
    - WLAN
    - Mobile IP
    - RFID
    - Wireless Sensor Networks
categories:
    - Security
    - Wireless Network
---

《无线网络与物联网安全》后半部分大纲，主要介绍了GSM，3G，WLAN，RFID，Mobile IP和Wireless Sensor Networks等方面的安全机制。这也只能作为一个临时大纲，后面回过头来看，大概自己也看不懂了⊙﹏⊙b汗，毕竟有太多的内容需要补充。

<!--more-->

# GSM Security

- basic security goals & aspects
- SIM card
	- IMSI
	- PIN
	- TMSI
	- K<sub>i</sub>
	- K<sub>c</sub>
	- LAI
	- List of last call attempts
- Tasks of Components
	- AUC
	- HLR
	- VLR
- Key Management Scheme
	- K<sub>i</sub>
- Authentication
	- Authentication goals
	- Authentication Scheme:Challenge-Response
	- Authentication Parameters
	- Authentication Protocol
		- Implementation of A3 and A8
		- Comp128
- Confidentiality
	- A5 stream cipher
	- K<sub>c</sub>
	- FN:Frame Number
- Anonymity
	- TMSI
	- Subscriber Identity Protection
- Detection of Compromised Equipment
	- IMEI
	- EIR
- Attacks against GSM security
	- anonymity
	- authenticity and confidentiality
    
# 3G Security

- Problems with GSM Security
- 3GPP Architecture - UMTS
	- RNC
	- USIM
- 3GPP Security Principles
	- Mutual Authentication
	- Integrity protection of signalling message
	- Use of stronger encryption
	- key freshness
- 3GPP Security Architecure
	- AN, HE, SN, ME, USIM
- Security Feature Groups
	-  Network access security
		-  User identity confidentiality
			-  IMSI
			-  TMSI
			-  VLR
		-  Entity authentication
			-  UMTS Authentication
		-  Confidentiality
		-  Data Integrity
		-  Mobile equipment identification
	-  Network domain security
		-  UTMS Radion Access Link Security
	-  User domain security
		-  USER-to-USIM authentication
		-  USIM-Temincal Link
	-  Application domain security
	-  Visibility and configurability of security 
- UMTS Authentication
	- SQN
	- RAND
	- AMF
	- K
	- MAC
	- XRES
	- CK
	- IK
	- AK
	- AV
	- AUTN
	- f1 f2 f3 f4 f5
- Data Integrity 
	- f9
	- IK
	- COUNT-1
	- FRESH
	- DIRECTION
- Ciphering Method
- Problems with 3G Security

# WLAN Security

- WEP(Wired Equivalent Privacy)
	- Security goals
		- Access Control
			- Challenge-Response
		- Data Confidentiality
			- RC4 Algorithm
				- Key Scheduling Algorithm
				- Pseudo-Random Generation Algorithm
		- Data Integrity
			- CRC->ICV
	- WEP Problems
		- Access Control
			- Authentication is one-way 
			- same shared secret key for authentication and encryption
		- Integrity
			- possible for an attacker to flip selected bits of message
		- Confidentiality
			- IV reuse and weak key
		- Key Management
- 802.11i
	- Protocol Architecture
	- Authentication
	- access control model
	- improved key management
	- TKIP
	- AES-CCMP
		- integrity protection  CBC-MAC
		- encryption CTR mode
		
# Mobile IP Security

- Moblie IPv4
	- Entities
		- Home Agent
		- Mobile Node
		- Foreign Agent
		- Correspondent Node
	- Terms
		- Care-of-Address
	- Problems with MIPv4
		- Suboptimal "triangle" routing
- Moblie IPv6
	- Binding Updates
	- False Binding Updates
	- BU Authentication
		- v1 Flooding Attack
		- v2 Exhausting State Storage
		- v3 Amplification 
	- Design Principles
		- Return routability
		- Resilience against DoS attacks
		- Strong authentication
		- Balancing message flows

# RFID Security

- Hash Locks
	- authenticate reader to the RFID tag
	- Randomized Hash Locks
- HB Protocol
	- authenticate RFID to the reader
	- HB Protocol-LPN
- HB+ Protocol
- How does the Reader read a Tag
	- Tree Walking

# Key Management in Wireless Sensor Networks

- Key Pre-distribution
	- Master-key Approach
	- Pair-wise Key Approach
- Probabilistic Key-sharing
- Random Graph
	- Erdos-Renyi formula
		- P<sub>c</sub> 
		- n
		- p
		- d
	- considering transmission range
		- P<sub>c</sub>
		- d
		- n<sup>'</sup>
		- p<sup>'
		- n(P)
		- k
	
## Pairwise Key Pre-Distribution 

- Eschemauer-Gligor Scheme
- Blom Scheme
	- Public matrix G
	- Private matrix D (sysmmetric)
	- λ-secure
	- Multiple Space Scheme
		- τ
		- ω
- Polynomial-based approach
	- Setup(Pre-distribution)
	- Direct Key Establishment
	- Path Key Establishment

## False Data Filtering

- Non-deterministic approach: SEF
	- En-route Filtering
	- Bloom Filters
- Deterministic approach: IHA
	- Association Discovery
	- Report Endorsement
	- En-Route Filtering
- Mixed approach: LEDS


