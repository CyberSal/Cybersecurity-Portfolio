#  Network Reconnaissance

**Author:** CyberSal  
**Authorization:** ✅ Performed in an **EC-Council iLabs** sandbox and explicitly authorized for educational/assessment use.

## Scenario 🛡️
I simulated an attacker's enumeration phase to audit our environment's defensive posture.

## Objectives 🎯
- Check for live systems and open ports  
- Identify active services on those systems  
- Perform light banner/OS fingerprinting  
- Note potential vulnerabilities / next steps

## Tools Used 🧰
- **Nmap** (CLI) and **Zenmap** (GUI)
- Parrot Linux attacker VM
- Screenshot capture for evidence

---

## Lab Environment (summary) 🧪
- Attacker host: Parrot Linux + Zenmap GUI  
- Target: `10.10.1.22` (Windows Server with AD roles and common services)

---

## Method & Results 📊

### 1) SYN (Half-Open) Scan — Fast service discovery
**SYN scan (-sS)**  
`nmap -sS -v 10.10.1.22`  
![SYN scan](./Screenshots/01_Nmap_sS_scan.png)

**TCP connect scan (-sT)**  
`nmap -sT -v 10.10.1.22`  
![Connect scan](./Screenshots/01_Nmap_sT_scan.png)

**Topology** (Zenmap → Topology)  
![Topology](./Screenshots/01_Zenmap_Topology.png)

**OS fingerprint**  
`nmap -O -v 10.10.1.22`  
![OS fingerprint](./Screenshots/01_Nmap_OS_Fingerprint.png)


---

## Quick Notes 📝
- Multiple services typical of a Windows AD host were detected (e.g., Kerberos, LDAP, SMB).  
- Findings will guide targeted enumeration in the next lab (SMB shares, LDAP objects, SNMP, etc.).  

---

---

## Report 📑

### Executive Summary
Active reconnaissance of `10.10.1.22` identified a Windows Server running typical **Active Directory** services (Kerberos, LDAP/LDAPS, SMB), plus **RDP**, **DNS**, and **MSMQ**. These services collectively suggest a domain controller (or a member server with AD roles) and present multiple enumeration paths for identity and access reviews.

### Scope & Approach
- Host scanned: `10.10.1.22`
- Techniques: Nmap **SYN** scan (`-sS`), **TCP connect** scan (`-sT`), OS fingerprinting (`-O`), and **Zenmap Topology** view
- Evidence captured as screenshots and linked inline

---

## Findings 🔎

### Open Services (Observed)
| Port | Service (common name) | Likely Role / Notes |
|-----:|------------------------|---------------------|
| 53/tcp  | domain (DNS)              | AD-integrated DNS (name resolution, zone data) |
| 80/tcp  | http                      | Web / management endpoint (check for server headers & apps) |
| 88/tcp  | kerberos-sec              | AD Kerberos (ticketing/authN) |
| 135/tcp | msrpc                     | RPC endpoint mapping (used by many Windows mgmt ops) |
| 139/tcp | netbios-ssn               | Legacy SMB over NetBIOS (share discovery, name svc) |
| 389/tcp | ldap                      | AD LDAP (directory queries) |
| 445/tcp | microsoft-ds (SMB)        | SMB/CIFS file & named pipe access |
| 464/tcp | kpasswd5                  | Kerberos password service |
| 593/tcp | http-rpc-epmap            | RPC over HTTP |
| 636/tcp | ldaps                     | LDAP over TLS (secure directory queries) |
| 3268/tcp| globalcatLDAP             | Global Catalog (forest-wide searches) |
| 3269/tcp| globalcatLDAPssl          | Global Catalog over TLS |
| 3389/tcp| ms-wbt-server (RDP)       | Remote Desktop |
| 1801/tcp| msmq                      | Microsoft Message Queuing |
| 2103/2105/2107 | zephyr/msmq-mgmt   | MSMQ mgmt & messaging endpoints |

> **Evidence:**  
> - **SYN scan:** `./Screenshots/01_Nmap_sS_scan.png`  
> - **TCP connect scan:** `./Screenshots/01_Nmap_sT_scan.png`  
> - **OS fingerprint:** `./Screenshots/01_Nmap_OS_Fingerprint.png`  
> - **Topology:** `./Screenshots/01_Zenmap_Topology.png`

### Risk-Oriented Notes
- **AD core services (Kerberos/LDAP/SMB/GC):** enable rich **identity enumeration** (users, groups, SPNs). Weak policy or legacy protocols can lead to credential abuse or relay paths.  
- **RDP (3389):** direct interactive access path; review NLA/MFA, lockouts, and allowed groups.  
- **SMB (139/445):** potential for **share enumeration**, sensitive file discovery, or legacy signing settings.  
- **LDAP/LDAPS (389/636/3268/3269):** directory data exposure if anonymous or low-priv binds disclose too much.  
- **DNS (53):** zone transfers or misconfig may disclose internal structure.  
- **MSMQ (1801/210x):** often overlooked; occasionally useful for lateral movement or info disclosure.

---

## Next Steps 🚀

> 🟣 Next Steps: Purple Team Integration & Validation

The core goal of this next phase is to close the identified security gaps and immediately **validate** that the new controls successfully block the Red Team's enumeration techniques.

### 1. Remediation & Hardening (Blue Team Focus)

The following steps address the critical misconfigurations that allowed the information leak:

* **SNMP (161/UDP):** The immediate priority is to **change the default community string (`public`)** to a complex, non-default value, and implement a **security access control list (ACL)** to restrict all SNMP traffic to authorized network management hosts only.
* **LDAP (389/TCP):** **Disable anonymous LDAP binds** on the Domain Controller. Additionally, enforce **LDAP Signing and LDAP Channel Binding** across the domain to mitigate Man-in-the-Middle (MITM) attacks and protect directory queries.
* **SMB (139/445):** Enforce **SMB signing** across the network via Group Policy and ensure all shares are configured using the **Principle of Least Privilege**.

### 2. Detection Engineering (Blue Team Focus)

The team will develop and test new detection rules in the SIEM based on the Red Team's confirmed attack paths:

* **LDAP/AD Enumeration:** Create an alert rule to monitor for an unusual volume of **anonymous bind attempts** or successive failed login attempts using a high number of different usernames (indicating a password spraying attack using the harvested list).
* **SNMP Enumeration:** Implement network monitoring to alert on any unexpected traffic on **UDP Port 161** originating from non-management IP ranges.
* **SMB Reconnaissance:** Develop an alert for a high volume of failed **`smbclient -L`** (share enumeration) attempts from a single source IP within a short time frame.

### 3. Adversary Simulation & Validation (Purple Team Iteration)

The final step is to **re-run the exact attack commands** used in the enumeration phase to confirm that the mitigations (Pillar 1) and the detection rules (Pillar 2) are effective.

| Technique | Validation Test | Expected Outcome |
| :--- | :--- | :--- |
| **SNMP Access** | Re-run `nmap --script=snmp-sysdescr` | **FAILURE** (Service should not respond to `public` or should be filtered). |
| **Identity Harvesting** | Re-run `nmap --script=ldap-brute` | **FAILURE** (Anonymous bind should be rejected). |
| **SMB Share Mapping** | Re-run `smbclient -L //10.10.1.22 -N` | **FAILURE** (NULL/Anonymous session should be rejected). |

```bash
# Red Team Validation Test (Re-run): Expected to fail after GPO/ACL deployment
nmap -sU -p 161 --script=snmp-sysdescr 10.10.1.22
nmap -p 389 --script=ldap-brute 10.10.1.22
