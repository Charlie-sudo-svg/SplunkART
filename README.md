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

To start, I set up a folder called "folder1" (name doesn't really matter here) and I downloaded crowbar using **sudo apt-get install crowbar -y**
