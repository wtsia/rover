
# Contents
- [Contents](#contents)
- [Exam Objectives](#exam-objectives)
	- [SY0-601 Certification Exam Objective Map](#sy0-601-certification-exam-objective-map)
- [Chapter 1: General Security Concepts](#chapter-1-general-security-concepts)
- [Chapter 2: Router Security](#chapter-2-router-security)
		- [**Configuration Commands**](#configuration-commands)
		- [**Explanation of Commands**](#explanation-of-commands)
	- [Vulnerability Management](#vulnerability-management)
		- [](#)

# Exam Objectives
## SY0-601 Certification Exam Objective Map
| Objective | Chapter |
| :---: | :---: |
| 1.0 Threats, Attacks and Vulnerabilities | |
| 1.1 Compare and contrast different types of social engineering techniques | Chapter 4 |
| 1.2 Given a scenario, analyze potential indicators to determine the type of attack | Chapters 3, 4, 7 |
| 1.3 Given a scenario, analyze potential indicators associated with application attacks | Chapters 6, 12 | 
| 1.4 Given a scenario, analyze potential indicators associated with network attacks | Chapters 3, 12, 13 | 
| 1.5 Explain different threat actors, vectors, and intelligence sources | Chapter 2 | 
| 1.6 Explain the security concerns associated with various types of vulnerabilities | Chapters 1, 2, 5 | 
| 1.7 Summarize the techniques used in security assessments | Chapters 5, 14 | 
| 1.8 Explain the techniques used in penetration testing | Chapter 5 | 
| 2.0 Architecture and Design | | 
| 2.1 Explain the importance of security concepts in an enterprise environment | Chapters 1, 6, 7, 9, 10, 11, 12 | 
| 2.2 Summarize virtualization and cloud computing concepts | Chapter 10 | 
| 2.3 Summarize secure application development, deployment, and automation concepts | Chapter 6 | 
| 2.4 Summarize authentication and authorization design concepts | Chapter 8 | 
| 2.5 Given a scenario, implement cybersecurity resilience | Chapter 9 | 
| 2.6 Explain the security implications of embedded and specialized systems | Chapter 11 | 
| 2.7 Explain the importance of physical security controls | Chapters 9, 15 | 
| 2.8 Summarize the basics of cryptographic concepts | Chapters 7, 11 | 
| 3.0 Implementation | | 
| 3.1 Given a scenario, implement secure protocols | Chapter 12| 
| 3.2 Given a scenario, implement host or application security solutions | Chapters 6, 11 | 
| 3.3 Given a scenario, implement secure network designs | Chapter 12 | 
| 3.4 Given a scenario, install and configure wireless security settings | Chapter 13 | 
| 3.5 Given a scenario, implement secure mobile solutions | Chapter 13 | 
| 3.6 Given a scenario, apply cybersecurity solutions to the cloud | Chapter 10 | 
| 3.7 Given a scenario, implement identity and account management controls | Chapter 8 | 
| 3.8 Given a scenario, implement authentication and authorization solutions | Chapter 8 | 
| 3.9 Given a scenario, implement public key infrastructure | Chapter 7 | 
| 4.0 Operations and Incident Response | | 
| 4.1 Given a scenario use the appropriate tool to assess organizational security | Chapters 4, 5, 11, 12, 15 | 
| 4.2 Summarize the importance of policies, processes, and procedures for incident response | Chapter 14 | 
| 4.3 Given an incident, utilize appropriate data sources to support an investigation | Chapter 14 | 
| 4.4 Given an incident, apply mitigation techniques or controls to secure an environment | Chapter 14 | 
| 4.5 Explain the key aspects of digital forensics | Chapter 15 | 
| 5.0 Governance, Risk, and Compliance | | 
| 5.1 Compare and contrast various types of controls | Chapter 1 | 
| 5.2 Explain the importance of applicable regulations, standards, or frameworks that impact organizational security posture | Chapters 10, 16 | 
| 5.3 Explain the importance of policies to organizational security | Chapter 16 | 
| 5.4 Summarize risk management processes and concepts | Chapter 17 | 
| 5.5 Explain privacy and sensitive data concepts in relation to security | Chapter 17 | 


# Chapter 1: General Security Concepts
Question: What are the Security Control Categories?
- enables **CIA triad (Confidentiality, Integrity, Availability)** + non-repudiation
- POMT: Physical, Operational Managerial, Technical -- Please Offer Me Tea
- Operational: Controls human element like policies, procedure, training. 
Conclusion: POMT

Q: Sec. Control Function Types?
- Goal/Function performed
- Identity Access Management--IAM
- Prevention, Detection, Correction
	- Before, During, After
	- Access Control Lists, Logs, Backup system
- + Directive, Deterrent, Compensating
	- Training and awareness prog, signs of warnings, alternative to primary
	- Ex of Compensating: legacy system cant encrypt at rest, so restrict access
C: The main ones are Prevention, Detection and Correction, with Directive, Deterrent, and Compensating as lesser mentioned types. 

What is Zero Trust?
- Adaptive Identity, Thread Scope Reduction, Policy-Driven access control 
- Control vs Data

# Chapter 2: Router Security
Question: What is ACL and how do you set it up?
- packet forwarding rules--can deny or allow traffic type/port
- standard/extended
	- filter  by ip/ip+socket
Conclusion: OSI Network layer. ACL 

Q: How do you secure a router
- change default settings
- disable UPnP
- apply ACLs
- anti-spoofing
A: Obfuscation and changing default settingts

Q: How do you configure an ACL on a Cisco Router?
```
# get into config, then create access-list
Router# conf t
# <command> <list no.> <ip to apply> <wildcard>
Router<config># access-list 10 deny 10.0.0.2 0.0.0.0


# applies ACL
Router<config># int fa0/1 
Router<config-if># ip access-group 10 in

# reverses the above for fast-ethernet
Router<config-if># no ip access-group 10 in

# deny icmp on the host from 10.0.0.2 to any (0.0.0.0)
Router<config># access-list 100 deny icmp host 10.0.0.2 0.0.0.0 255

# 
Router<config># access-list 100 permit ip 10.0.0.0 0.255.255.255 any
Router<config># int fa0/1
Router<config-if># ip access-group 100 in

```

A:


### **Configuration Commands**

This set of commands will create the standard access list, apply it to the VTY lines to filter incoming traffic, and save the configuration.

Bash

```
Router> enable
Router# configure terminal
!
! Create the Standard Access List
!
Router(config)# access-list 5 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 5 permit 192.168.2.0 0.0.0.255
Router(config)# access-list 5 permit 192.168.3.0 0.0.0.255
!
! Apply the ACL to the VTY lines
!
Router(config)# line vty 0 4
Router(config-line)# access-class 5 in
Router(config-line)# end
!
! Save the configuration
!
Router# copy running-config startup-config
```

---

### **Explanation of Commands**

- **`access-list 5 permit <network> <wildcard-mask>`**: This command creates the access list entries.
    
    - `5`: Specifies the access list number, which is in the range for a standard ACL (1-99).
        
    - `permit`: This is the action to take if the source IP matches the rule.
        
    - `0.0.0.255`: This is the **wildcard mask** for a `/24` network. It tells the router to only match the first three octets of the IP address (`192.168.1`) and ignore the last octet.
        
- **`line vty 0 4`**: This command enters the configuration mode for the virtual terminal lines, which are used for remote Telnet and SSH access.
    
- **`access-class 5 in`**: This applies access list number 5 to the VTY lines. The `in` keyword filters incoming connections, so the router will check the source IP of anyone trying to Telnet or SSH _into_ the router against the rules in ACL 5.
    
- **Implicit Deny**: All access lists have an invisible **`deny any`** rule at the end. This means any traffic that does not match one of the `permit` statements in ACL 5 will be automatically blocked, fulfilling the requirement to only allow access from the three specified networks.
    
- **`copy running-config startup-config`**: This command saves the currently active configuration (in RAM) to the startup configuration (in NVRAM), ensuring your changes persist after a reboot.

## Vulnerability Management
What are the categories of VM?
- Identification
	- Scans, Feeds, OSI, pentesting, disclosure, audits
- Threat Hunting
	- hypothesis -> investigation -> understanding
- Threat Analysis
	- CVSS (Common Vulnerability Scoring System)
	- CVE (Exposures)
- Threat Response/Remediations
	- Patching, Insurance, Segmentation, Compensating

End of Life?
- endoflife.date

## 