# Network Setup Virtualbox

All machines are connected via a NAT Network

### **Network Architecture**
| Component         | OS            | Role |
|------------------|--------------|------|
| **Domain Controller/Splunk Server** | Windows Server | AD DS, DNS, Sysmon, Splunk Server |
| **Workstation** | Windows 10 | Test user machine, Sysmon |
| **"Hacker"** | Kali Linux | Crowbar for brute forcing, Metasploit |
| **Vulnerable Target** | Windows 7 | Exploited by Metasploit |

![Screenshot 2025-02-19 211615](https://github.com/user-attachments/assets/ba6c2102-4b9a-4f24-a9d3-cf1b2f2ba9c0)
