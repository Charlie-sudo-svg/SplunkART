# SplunkART
Active Directory project with Splunk and Atomic Red Team


## **Lab Setup** 
The lab consists of the following machines & programs:

**Machines:**
- Kali Linux Machine with Crowbar installed
- Ubuntu Server to host Splunk server
- Windows Server 2022 as a Domain Controller
- Windows 10 Pro as the target machine

**Programs:**
 - Splunk for log analysis
 - Sysmon for log monitoring
 - Active Directory for user management
 - Atomic Red Team for testing security controls


## **Ubuntu Server**
To set up the Ubuntu server first I downloaded the ISO file: https://releases.ubuntu.com/jammy/. <-- Select server install image

Then after setting up the Ubuntu server with default configurations I did **Sudo apt-get update && Sudo apt-get upgrade**
I configured a static IP which was [192.168.10.10/24]

After this we need to access .deb file I installed on my host machine from https://www.splunk.com/en_us/download.html to install Splunk Enterprise on the Ubuntu Server. To do this, we need to set up a file share on the Ubuntu system. To do this, I used **Sudo apt-get install virtualbox-guest-utils** and **Sudo apt-get install virtualbox-guest-additions-iso**. After this I was able to successfully access the file share using virtualbox's GUI and the Ubuntu system. Once I got the package on my system I depackaged it onto my system and set it up using mostly default configurations. 

**Important Note:**
If you are doing this yourself I highly recommend running this command when your done setting up using the Splunk user, **Sudo ./splunk enable boot-start -user splunk** So that the server will start everytime you turn on the VM.

## **Windows 10 Pro Machine**
Lets begin setting up the target machine by installing Splunk Universal Forwarder to send our data from our machine to the Splunk server. To do this I went to https://www.splunk.com/en_us/download.html and downloaded the forwarder onto the machine and set it up. When on the recieving indexer tab I put the IP address as [192.168.10.10/24] using port 9997 as that is where we will be sending our telemetry.

The next step is to download and configure Sysmon as well as configure our inputs.conf file for our Splunk forwarder. I used Sysmon Olaf Config as my configuration file for Sysmon. I went onto powershell and ran **./Sysmon.exe -i ..\sysmonconfig.xml** <-- you can choose whatever config file you want I just used the Olaf one. 

Now we need to set up Splunk Forwarder with our Inputs.conf file by going into the local directory in the SplunkUniversalForwarder folder. I Used notepad and ran as administrator to configure the file as follows:

[WinEventLog://Application]
index = endpoint 
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false 
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

After saving the configuration file go to 'services' and restart the Splunk Forwarder service, also another note **here** go to properties for the Splunk service and go to the 'Log On' tab and change the Log on to **Local System Account**. 

We are almost done setting up Splunk! I went to the web browser and typed [192.168.10.10:8000] and logged in using my Splunk creditentials.

In Splunk, I went to settings and clicked Indexes and added the index **"Endpoint"**. Then I went to Forwarding and Recieving and configured the recieving port to be 9997. 

Splunk is now successfully set up!

![Screenshot 2025-02-05 125146](https://github.com/user-attachments/assets/a062c669-912c-4384-b00f-9adeababf384)


## **Windows Server 2022 Active Directory Setup**

I started off in the server manager app that comes preloaded in Windows Server 2022 and clicked **Manage** --> **Add Roles and Features** to enable Active Directory Domain Services. Once I enabled that option you have to promote your server to a domain controller. I selected **Add a new forest** and typed in the domain I wanted to use

![Screenshot 2025-02-05 165235](https://github.com/user-attachments/assets/53b2229b-7a52-4a5b-afb5-d5adc1685935)

I completed the other required items for the configuration the machine restarted and I logged in as CHARLIE/ADMINISTRATOR.

Once I logged in, I opened server manager and went to Active Directory, here I added 2 OUs, which where IT and HR to represent an IT department and an HR department. In each one I created one user and set a username and password to them.

When I added users I went back to the Windows 10 Pro machine to join it to the newly created domain. To do this I first had to open my network settings and change my preferred DNS server to [192.168.10.7] which is the static IP I set up for the Windows Server machine. I did this so the target machine can resolve the domain when joining it. 

Joining a domain from here is pretty simple, I went to system properties and did it from there. After a quick restart the target machine is finally a member of the domain!


## **Kali Linux Setup**

To start, I set up a folder called "folder1" (name doesn't really matter here) and I downloaded crowbar using **sudo apt-get install crowbar -y.** I then created a file called passwords.txt in folder1 and added a few random passwords, including an unsafe password used by my target machine. You can also choose to unzip rockyou.txt.gz in the /usr/share/wordlists directory.



## **Launching an attack!**

First step of this process required me to go back to the Windows 10 machine and enable port 3389 or RDP. I went to "About PC" and clicked on properties then went to advanced system settings to enable rdp for both of my Active Directory users.

Next, I went back to the Kali Linux machine to use Crowbar. Using Crowbar -h is very helpful to find information on what command we should next use, I'll link the tool below if you want to read and use it for yourself... Safely of course! 

https://www.kali.org/tools/crowbar/

Using the documentation I ran the command **crowbar -b rdp -u jsmith -C passwords.txt -s 192.168.10.100/32**. A note here is that I had to include CIDR notation when doing this command or else it would not have worked.

![Screenshot 2025-02-06 001558](https://github.com/user-attachments/assets/73348085-d5af-4670-b4a5-c4a5dc83ef41)

Checking Splunk reveals interesting telemetry generated by running crowbar:

![Screenshot 2025-02-06 004038](https://github.com/user-attachments/assets/79e17f95-66c5-4a6f-911d-3ac89c12e4e1)


## **Setting up Atomic Red Team**

Going back to the Windows 10 machine I added an exclusion for the C drive for the purposes of the test. I typed in this command to install Atomic Red Team: **IEX (IWR https://githubusercontent/redcanaryco/invoke-atomicredteam/master/installatomicredteam.ps1 -UseBasicParsing** then **Install-AtomicRedTeam -getAtomics**

Now that the installation is complete I had a bit of fun running some sample telemetry. I ran **Invoke-AtomicTest T1136.001** which creates a new user account. I was then able to capture that telemetry onto Splunk, where it can be used for further analyzing. 


## Conclusion

This project successfully demonstrates how to integrate Splunk with Active Directory and Atomic Red Team for monitoring and analyzing cybersecurity events. By leveraging Splunk for log aggregation, Sysmon for detailed event logging, and Atomic Red Team for simulating attacks, we've created a lab environment capable of detecting malicious activities and tracking security incidents in real-time.

The attack simulations with Crowbar and the use of Atomic Red Team tests provide valuable insights into how security controls can be tested and monitored in an enterprise network. The data collected by Splunk from these tools offers a powerful way to understand the behavior of threats and help cybersecurity professionals respond faster and more effectively.

This setup can serve as a foundation for a more advanced Security Operations Center (SOC) simulation, providing hands-on experience with the tools and techniques used in modern cybersecurity environments.
