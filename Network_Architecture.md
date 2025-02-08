# Network Setup Virtualbox

All machines are connected via a NAT Network

### **Network Architecture**
| Component         | OS            | Role |
|------------------|--------------|------|
| **Domain Controller** | Windows Server | AD DS, DNS, Sysmon |
| **Workstation** | Windows 10 | Test user machine, Sysmon |
| **SIEM Server** | Ubuntu | Splunk for log analysis |
| **"Hacker"** | Kali Linux | Crowbar for brute forcing |

![Screenshot 2025-02-07 225747](https://github.com/user-attachments/assets/f3041f47-bbab-4f8f-a93f-2f252b74c42e)
