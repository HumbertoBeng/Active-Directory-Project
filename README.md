![image](https://github.com/user-attachments/assets/0682bdd5-7937-4a5f-be25-d07d90843779)# Active-Directory-Project

## Objective

This project is aimed to create an environment using Active Directory with the objective to learn how to setup this tool and how it can manage
Domains, Users, and machines. Also we will be using a Linux server tu create a Splunk SIEM instance to monitor the machines managed by the
Active Directory Domain so that we can generate telemetry and learn how different attacks look like in a SIEM environment, to achieve this we will alse be using
Kali Linux as an attacker machine using a Brute Force attack to attack the machines within the Active Directory Domain.



### Skills Learned
[Bullet Points - Remove this afterwards]

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used
[Bullet Points - Remove this afterwards]

- Splunk SIEM
- Sysmon
- Active Directory
- Atomic Red Team

## Steps

### Step 1.- Designing a Diagram

![imagen](https://github.com/user-attachments/assets/88f1be88-8f66-4d1e-aec1-7f2d196df9ca)


Explanation of the colored lines:

- The green dotted arrow going from our PC with Windows 10 to the Splunk Server indicates that it will forward data to Splunk.

- The green dotted line going from Active Directory Server to Splunk Server indicates that it will also forward data to Splunk.

### Step 2.- Configuring Virtual Machines

The VMs we'll be using are:
- Ubuntu Server with Splunk
- Windows Server 2022
- Kali Linux
- Windows 10

First of all, to ensure we have connectio between our Virtual Machines we are going to create a NAT Network in Virtual Box

2.1.- Setting up a NAT Network in Virtual box

To start creating a NAT Network in Virtual Box we are going to go to Tools and click on the bullet point and select the Network option. Refer to image 1.

![image](https://github.com/user-attachments/assets/20e8173b-ff33-431b-b7aa-4c31373008dc)
Image 1.

Once selected the Network option there'll be 3 tabs and we are going to select Nat Networks to create it. Refer to image 2.

![image](https://github.com/user-attachments/assets/b1296837-5591-458d-b530-b3b2686f7bd1)
Image 2.

Following the instructions of the diagram we made before, we are going to give our NAT Network a name, in this case we'll be using "AD-Project" as the name of our network, the ip will be of 192.168.10.0/24 and we'll leave checked the "Enable DHCP" option and finally click "Apply". Refer to image 3.

![image](https://github.com/user-attachments/assets/41056890-02dd-4031-9128-b6fba9aadae5)
image 3.

2.1.1.- Adding a Virtual Machine to a NAT Network

To add a Virtual Machine to a NAT Network we are going to go to the Virtual Machine we would like to add and left click it and select the "Settings" option. Refer to image 4.

![image](https://github.com/user-attachments/assets/4eb513fa-b850-492c-8722-fe828c2e93f8)
Image 4.

Once inside the Settings tab of our Virtual Machine we'll go to the "Network" section and change the network Adapter 1. For this we will change the option in the "Attached to:" from NAT to NAT Network. After selecting the NAT Network option under the "Attached to:" option the "Name:" option will let us choose the NAT Netowrk we'll be using in this Virtual Machine, so we'll change it to the name of the NAT Network we've created, in this case AD-Project and then click "Ok" to save the changes. Finally repeat the same process with the rest of the VMs we'll be using in this project. Refer to image 5.

![image](https://github.com/user-attachments/assets/1b05cefc-ace7-47d5-a977-1a5e616417ae)
Image 5.

2.2.- Configuring Ubuntu Server with Splunk

2.2.1.- Connectivity
First things first to start configuring our Ubuntu Server we need to check which IP does it have, so we are going to use the command "ip a". Since we checked the option of DHCP in the configuration of our NAT Network it will have an IP, but since in the first part of this project we set the IP address of the Ubuntu server to 192.168.10.10/24 we will need to change it, so to do this we'll need to edit a file to disable DHCP and change it to the IP we decided earlier.

So with this in mind, we'll head to the directory of /etc/netplan with the command "cd", after we click enter to the command, we'll look for a file called "00-installer-config.yaml". Sometimes this file would not be found in this directory, so for such cases we can just create said file by using "sudo nano 00-installer-config-yaml". This command will allow us to create the file and at the same time open a text editor to edit the content of said file.

Now once we open the file we'll need to write the next config:

![image](https://github.com/user-attachments/assets/20b3b2f9-6d52-40f8-bc54-43fbc1c7675d)
Image 6

A quick explanation of what is written inside this file, the "dhcp4: no" indicate that we will not be using the dhcp anymore so we'll need to deactivate it, and under that we have the "addresses: [192.168.10.10/24]" which refers to the IP address we would like the server to have.

Now after writing the configuration in this file, we can save it using CTRL + X and typing Y to save the changes. After saving the file we will run the command "sudo netplan apply". This command tells the netplan directory to apply the changes made to the files it contains. Refer to image 7.

![image](https://github.com/user-attachments/assets/580b5ec4-b944-47b5-b814-91c7da7103bb)
Image 7


Next we can see if the changes were applied by using the "ip a" command again. Refer to image 8.

![image](https://github.com/user-attachments/assets/f703ae37-ca77-48df-8e19-21e48c51741a)
Image 8


2.2.2.- Installing Splunk

To install Splunk in an Ubuntu server we will need to head to the Splunk and create an account to access to the downloads.

Once logged in with Splunk we'll head to the "Products" tab and select "Free Trials & Downloads". Refer to image 9

![image](https://github.com/user-attachments/assets/347e1359-1b4d-4ce7-b4ed-fc2c07e83b2c)
Image 9

Inside the "Free Trials & Downloads" page we'll look for Splunk Enterprise and select "Get My Free Trial" and then go for the Linux as our operating system and select the .deb file and click Download Now. Refer to image 10.

![image](https://github.com/user-attachments/assets/b7e1692d-3a18-435b-8a17-af82e50fb71a)
Image 10


Now that we've downloaded Splunk Enterprise we'll go back to our Virtual Machine to start installing the guest addon. To do that we'll use the command "sudo apt-get install virtualbox-guest-additions-iso". Refer to image 11

![image](https://github.com/user-attachments/assets/f7a71440-dd69-4f23-bc67-440f68dcffc0)
Image 11

once the installation has finished we can go and click the "Devices" tab within our Virtual Machine and head to "Shared folders" and then "Shared Folders Settings", once inside the configuration tab we will click the icon with the folder and with a "plus" sign which then will display a small window where we can select a "Folder Path:" and select the folder in which we saved the .deb file we downloaded from earlier, next the "Folder Name:" can be left as is, after that we will check al last 3 options and finally click "Ok". We do this so that we can use the .deb file we downloaded earlier which will allow us to configure Splunk. Refer to image 12.

![image](https://github.com/user-attachments/assets/813bf192-1ac9-484e-8bea-37a379ee6e6e)
Image 12


Following with the installation, we need to go back to our VM and reboot it, we can do so with the command "sudo reboot".

After rebooting the VM, our next objective will be adding our user to the vboxsf group so that we can access the folder with the files we just shared with this VM, and to do that we can type the command "sudo adduser [the name of your user] vboxsf". You may get the error "The group 'vboxsf' does not exist", and to fix that we need to install the guest, so to do that we can use the command "sudo apt-get install virtualbox-guest-utils" and then hit Enter. Once the command have been run, we reboot the VM again.

Now that we've installed the missing guests, we can try again the same process of adding our user to the vboxsf group. refer to image 13

![image](https://github.com/user-attachments/assets/fc5c4f1f-75fc-457e-b160-b94a7cd696f9)
Image 13.


Since we've added ourselves to the group "vboxsf" we need to create a new directory called "share". We'll use the command "mkdir share" to perform this action.

Afterwards we'll made the connection to the folder we added where we downloaded Splunk file to the folder we just created in our Virtual Machine. To do that we'll use the command "sudo mount -t vboxsf -o uid=1000,gid=1000 [The name of the folder where you saved Splunk] share/". After that we can check if the command was run succesfully by accessing the "share" folder. 

Finally, to install Splunk we'll have to be inside our "share" folder and run the next command: "sudo dpkg -i [the name of the Splunk installer.deb]" and after running the command, once we see a "complete" we should be good to go. So now that we've installed Splunk in our Ubuntu Server we can access the Directory created that can be found in /opt/Splunk. Inside the directory where Splunk is located we can see that the permissions of the files inside said Directory are for a user called "splunk". Refer to image 14.

![image](https://github.com/user-attachments/assets/3df762bd-1005-48c4-bbc8-a45abc3ca179)
Image 14

So to have permission to use those files we need to change our user to "splunk" using the following command: "sudo -u splunk bash", notice that after using the command the user change from your original user to "splunk". Refer to image 15.

![image](https://github.com/user-attachments/assets/57ebf925-6db6-490d-ad52-e7c72cfbd637)
Image 15.


Accessing the Directory of "bin" would allow us to see all the binaries used by Splunk. Once inside the "bin" Directory we will use the "./splunk" binary and use it along the command "start". After reading the terms and conditions we select "Yes" and set the administrator username the same as our original user and then a super secure password. Finally once the process has finished we use "exit" to exit the splunk user. After returning to our original user we enter the "/opt/splunk/bin" Directory and run the following command: "sudo ./splunk enable boot-start -user splunk". This command allow the server to start Splunk with the user "Splunk" everytime the server reboots.

2.3.- Configuring the target machine and the Windows Server machine

We are going to be configuring our Windows 10 machine as the target machine for this project. The instructions for the Windows Server machine will not be written here since they are pretty much the same as the one for our Windows 10 machine.

First we are going to need to change the name of the Windows 10 machine to "Target-PC", to do that we can search for PC -> Right click and select Properties -> Rename This PC. Refer to image 16.

![image](https://github.com/user-attachments/assets/c0ca6c7d-26c6-431b-bdcb-59d72352ef48)
Image 16

Once we've renamed our Windows 10 machine we are going to restart it. After restarting our Virtual Machine we can make sure if the changes were applied by searching for PC -> Right click and select Properties

Now that we've changed the name of our target machine, we need to check if the IP is the same as the one we assigned in our Diagram in the first part. So to see what the IP of our machine is we can go to CMD and use the command "ipconfig". Refer to image 17.

![image](https://github.com/user-attachments/assets/58ca9c7a-73bc-49da-921c-6f75219fadc7)
Image 17

After we ran the command we can see that the IPs don't match. To change it we can right click the network icon in the right down corner of our screen and select "Open Network & Internet settings". Refer to image 18.

![image](https://github.com/user-attachments/assets/b9d3f5ee-171a-4bf5-b3e7-4fb3a2ec8e32)
Image 18

Once a screen with the options of our network appears, we'll select "Change adapter options", then right click the adapter and select "Properties", afterwards inside the "This connection uses the following items:" section we need to locate the "Internet Protocol Version 4 (TCP/IPv4)" option and double click it, change the options from "Obtain an IP address automatically" to "Use the following IP address" to write a static IP. At the end it should look like this. Refer to image 19.

![image](https://github.com/user-attachments/assets/210f7a87-54d4-47f7-9401-3c3a12c030c8)
Image 19


Once we've saved the configuration, we can make sure if the changes were saved by using CMD and writing the command "ipconfig" again. Refer to image 20.

![image](https://github.com/user-attachments/assets/7e429538-4693-4fd3-b3ff-87571202c220)
Image 20

2.3.1.- Installing Splunk Forwarder and Sysmon 

2.3.1.1.- Installing Splunk Forwarder

The Splunk forwarder allows us to forward all the telemetry generated in the machine in which is installed directly to our SIEM.

So to start with the installation we should go to the official page of Splunk where we will login with our account and then select the Products dropdown and go for "Free Trials & Downloads". Once inside the page scroll down a little bit until you see the "Universal Forwarder" and click the "Get My Free Download". Now we should download the 64-bit .msi file. Refer to image 21.

![image](https://github.com/user-attachments/assets/302a2014-2451-4724-9e5a-3a14b7abefe9)
Image 21.

Once we have downloaded the file we can go to the folder in which we saved said file and double click it.


To install the Universal forwarder, first and foremost we are going to accept the license agreement and make sure we have selected the option of "An on-premise Splunk Enterprise instance" and then click next. Refer to image 22.

![image](https://github.com/user-attachments/assets/c7f666e2-1b63-434d-8d61-1a71550f0bec)
Image 22

Now for the username we can choose whatever name, in this case we'll be using "Admin" and leave the "Generate random password" checked and click next. Refer to image 23.

![image](https://github.com/user-attachments/assets/fea8848d-2b04-42ef-a97d-3dffc52f90a9)
Image 23

We can skip the next screen. For the next screen, the Receiving Indexer we are going to write our Splunk Server IP and the port we will use the default one and Finally click Next one more time and finished the installation. Refer to image 24.

![image](https://github.com/user-attachments/assets/a6feadca-76a9-4455-a93e-9450e70e4e1f)
Image 24


2.3.1.2.- Installing Sysmon

While the Universal Forwarder is installing we can go ahead and install the Sysmon from Sysinternals. Also we'll be using the Sysmon configuration from olafhartong from its Github page. Refer to image 25.

![image](https://github.com/user-attachments/assets/b0489627-50ba-4461-9e3d-18e22f9de12d)
Image 25

Once inside Github we can scroll down until we see sysmonconfig.xml and we'll click it, select the raw content and finally right click it and save it. Refer to image 26, 27 and 28.

![image](https://github.com/user-attachments/assets/657fb506-9c7f-48c5-b054-3992bcabfcc1)
Image 26

![image](https://github.com/user-attachments/assets/60a3face-e5af-46ca-9754-0164cafa8f06)
Image 27

![image](https://github.com/user-attachments/assets/12080e71-3daf-4f4d-81df-8a5d442f3569)
Image 28


To install Sysmon we need to unzip the file we downloaded from Sysinternals, so once done that we are going to execute from the command line so we'll need to copy the path to the folder with the Sysmon files. Refer to image 29.

![image](https://github.com/user-attachments/assets/01f607e2-8c14-4a59-9732-17d47f78db87)
Image 29

So following the installation we are going to open PowerShell with administrative privileges and then change directories to the path we just copied. Inside the path we are just going to execute Sysmon64 using the following command command: ".\Sysmon64.exe -i ..\sysmonconfig.xml" the "-i" in this command indicate the configuration file that Sysmon is going to use. A small window will appear we just need to hit Agree and the installation will start. Refer to image 30.

![image](https://github.com/user-attachments/assets/2ede4836-763e-4d7f-8023-b3271f672ad3)
Image 30

2.3.2.- Instructing our Splunk Forwarder

Now that we have installed both Splunk Universal Forwarder and Sysmon, we need to create a file that will instruct our Universal Forwarder which data needs to be send over to our Splunk Server, and to do that we must configure a file called "inputs.conf" which is located in "C:\Program Files\SplunkUniversalForwarder\etc\system\default". Refer to image 31.

![image](https://github.com/user-attachments/assets/a6c7ce67-853f-4287-9b9b-7670c551913d)
Image 31

This is very important, you don`t want to mess with this file, instead we are going to create a copy of said file under the "local" directory, but it has a catch, to modify anything inside this directories we need Administrative privileges, so we are going to create a new inputs.cof file by running the notepad with Administrative privileges and write the following. Refer to image 32:

![image](https://github.com/user-attachments/assets/2601fe25-200f-4ad5-92b2-24a4ef176240)
Image 32

Finally we will save this file to the path we mentioned before and rename the file to "inputs.conf" -This configuration is instructing our Splunk Forwarder to send over events related to Application, Security, System and Sysmon to our Splunk Server. Refer to image 33.

![image](https://github.com/user-attachments/assets/7683f32b-9a8d-41c2-acb4-bf154d5b052f)
Image 33

As a side note, every time you update the "inputs.conf" file you'll need to restart the Splunk Universal Forwarder service, and to do that we need to search for "Services" and run it as an Administrator. Once executed we can just search for the "Splunkforwarder" service and double click it, access the "Log On" tab and change the account to "Local System Account" otherwise the Splunkforwarder won't be able to retrieve events and forward them. Refer to image 34.

![image](https://github.com/user-attachments/assets/8a15ac09-8e2d-40ca-998e-45d578d029e0)
Image 34

For the changes to be applied we'll need to restart the service, and to do that we can right click the service and select "Restart". Sometimes you will receive a warning message saying that the Service could not be stopped, and thats fine you can just click "Ok" and click "Start the service" in the left side. Refer to image 35.

![image](https://github.com/user-attachments/assets/bbb61995-93cb-4b94-9b10-95500ce16e6a)
Image 35

2.3.3.- Configuring Splunk SIEM

We want to configure our Splunk SIEM to see the telemetry generated by our target machine, to do that we can open a web browser and type the IP of our Splunk server with the port 8000 so we can access to Splunk.
Image 36.

![image](https://github.com/user-attachments/assets/ead7812a-954b-4662-9749-8997e7779bcb)
Image 36.

The Username and Password are the same we used to create our Splunk Server.


Once we login, we need to add the index of the Splunk Universal Forwarder. So to do that we can go to "Settings" at the top of the page, and look for "Indexes". Refer to image 37.

![image](https://github.com/user-attachments/assets/152bb54d-5b4a-4391-a856-857d5d1d6e5f)
Image 37


To create the new index, we need to click on the "New Index" button and the upper right corner and Once a window is displayed prompting us with giving it a name, we will name it "endpoint" and hit "Save".The reason for the name is that is the same name the index in our "inputs.conf" file has. Refer to image 38.

![image](https://github.com/user-attachments/assets/7e773717-36a6-49a9-abab-a10aa8ae6d05)
Image 38


Next we need to enable our Splunk Server to receive the data. To do that we need to go to "Settings" again and then click "Forwarding and receiving". Once inside, under the "Receive data" section we are going to click on "Configure receiving", then "New Receiving Port" and for the port we are going to use the 9997 as that is the default port and the same we input in the Splunk Universal Forwarder and then click Save.

If everything went right we should be seeing events being generated in the index of "endpoint". So to verify that we can go to "Apps" at the top of the page, and select "Search and reporting" and under the Search bar we'll type "index=endpoint" and then click search. If the installation was successful we should be seeing a lot of events already generated. Refer to image 39.

![image](https://github.com/user-attachments/assets/8a886f89-b5ed-43ce-ad59-e4e526113211)
Image 39



















































































































