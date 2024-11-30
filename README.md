![image](https://github.com/user-attachments/assets/840c74a4-63e7-40dd-b3c8-2d98e2631c9c)# Active-Directory-Project

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

### Step 2.- 

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


Accessing the Directory of "bin" would allow us to see all the binaries used by Splunk. Once inside the "bin" Directory we will use the "./splunk" binary and use it along the command "start". After reading the terms and conditions we select "Yes" and set the administrator username the same as our original user and then a super secure password. Finally once the process has finished we use "exit" to exit the splunk user. After returning to our original user we enter the "bin" Directory and run the following command: "sudo ./splunk enable boot-start splunk". This command allow the server to start Splunk with the user "Splunk" everytime the server reboots.





























































































































