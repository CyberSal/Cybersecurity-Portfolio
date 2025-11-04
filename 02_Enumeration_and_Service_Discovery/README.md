# System Enumeration & Service Discovery

This project expands on reconnaissance by performing deep enumeration against discovered systems.  
It covers extracting information from network services such as SMB, SNMP, LDAP, and DNS to understand infrastructure and user exposure.

**Key Tools:** Enum4linux, Nmap NSE scripts, SNMPwalk, DNSrecon, ldapsearch  
**Skills Demonstrated:** Account enumeration, share discovery, service version detection, and privilege audit preparation.
# Project 02 – System Enumeration & Service Discovery

**Objective:**  
Perform detailed enumeration against discovered hosts to extract service, share, and directory information that supports targeted vulnerability analysis.

---

**Tools & Environment**
- Kali Linux (VM)  
- enum4linux | snmpwalk | ldapsearch | smbclient | rpcclient

---

 **Steps Performed**
1. Executed `enum4linux -a` to enumerate SMB shares, users, and server details.  
2. Queried SNMP for system and interface information using `snmpwalk`.  
3. Performed LDAP queries to collect directory structure and user objects.  
4. Compiled raw outputs into `/Results/` and captured screenshots in `/Screenshots/`.

---

**Evidence**
| File | Description |
|------|-------------|
| `Results/enum4linux_output.txt` | Raw SMB/NetBIOS enumeration output |
| `Results/snmpwalk_output.txt` | SNMP device info and interface listing |
| `Results/ldapsearch_output.txt` | LDAP directory output (if applicable) |
| `Screenshots/01_enum4linux_output.png` | enum4linux terminal screenshot |
| `Screenshots/02_snmpwalk_output.png` | snmpwalk terminal screenshot |
| `Screenshots/03_rpc_smb_output.png` | rpcclient/smbclient output screenshot |

---

 **Key Findings**
- Shares and service accounts discovered on Windows host `10.10.10.12`.  
- SNMP responded to default community string `public` on network device `10.10.10.30`.  
- Anonymous LDAP bind returned user objects — follow up required.

---

**Next Steps**
- Move to targeted vulnerability scanning of identified services (Project 03).  
- Conduct password audits and access control reviews for discovered shares/accounts.

*Completed by Salmata Lamin*
