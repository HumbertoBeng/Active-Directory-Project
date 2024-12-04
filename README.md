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



### Step 3.- Installing and Configuring Active Directory

In this step we are going to be Installing and Configuring Active Directory in our Windows Server, then we'll promoted to be a Domain Controller and at the end we'll configure our target machine to join the Domain.

3.1.- Installing Active Diractory in our Server

So to start the installation of Active Directory, we are gonna need to go to our Windows Server Machine and Open "Server Manager". Once inside the program, we are gonna look for the option "Manage" at the top right corner and then click "Add Roles and Features". Refer to image 40.

![image](https://github.com/user-attachments/assets/c9b5da5b-1328-4567-a88a-75076922c222)
Image 40

Now a window will appear, we need to click next in the first part, then in "Installation Type" we need to make sure to select the option called "Role-based or feature-based installation" and then select Next. In the Server Selection is where all the servers we have would appear, but since we are only using one, only that would be showed.  section we would need to click Next again. In the "Server Roles" section of the installation wizard, we are going to need to check the "Active Directory Domain Services, a window will appear and we just need to click "Add Features". Once we've added the features, we can just go ahead and click Next in the next sections and at the end click Install. Refer to image 41.

![image](https://github.com/user-attachments/assets/2ac96af0-523a-4fec-a90a-54d9d7cd2337)
Image 41


What we want to do next is to close the installation wizard and go to the main page of the Server Manager. Then at the top we need to click the Flag with a warning icon next to it, this is the notification button. Once we've open it a message of "Post-deployment Configuration" would appear with an option highlighted called "Promote this server to a domain controller" and we need to click it. Refer to image 42. 

![image](https://github.com/user-attachments/assets/33ace5e3-1527-468b-a5df-a542a4f153da)
Image 42


A new window will appear. To start configuring our server as a Domain Controller, in the first section called "Deployment Configuration" we need to select the "Add a new forest" option, this is because we want to create a brand new domain. As for the Domain Name, we can name it however we want. In this case I'm going to name it csbarista.local. The Domain Name must have a top level domain, so it can't be just "csbarista", it needs to be "csbarista.*something*". once we've decided which name we want our server to have, click Next. Refer to image 43.

![image](https://github.com/user-attachments/assets/43c487a8-daeb-4c90-ab20-e131f36b563d)
Image 43


In the next section leave everything default and put it a super secure password and then keep clicking Next until the "Paths" section. Here are the paths used to store our database file named NTDS.DIT which contains everything from the Active Directory, including password hashes. Moving forward, just keep clicking Next until you get the Install button. Refer to image 44.

![image](https://github.com/user-attachments/assets/34cdc710-5d12-42ba-9739-c082c51d7698)
Image 44

Once the installation finishes the server is going to restart.


Now that the server has restarted and applied the configurations, noticed that that a new user has appeared in the logging screen. This means that we successfully promoted our server to a Domain Controller. The next step would be creating some new users.

In our Server Manager, we are going to select Tools in the top right corner and then select "Active Directory Users and Computers". Refer to image 45.

![image](https://github.com/user-attachments/assets/7d177a6a-2131-4cc6-9fe2-50b61a695531)
Image 45

This is where we can create new objects such as users, groups and many more. Now go ahead and expand our Domain in the left part of the window. The folder called "Builtin" contains groups that have been automatically created by Active Directory. To learn more of what each group has permissions to, we can just go ahead and double click any group. The folder named "Users" contains a list of Users and some additional groups. Refer to image 46.

![image](https://github.com/user-attachments/assets/c5b0deed-d78b-4090-87bf-2be7eeda393e)
Image 46

Now we are going to create a "Organization" within our Active Directory, to do that we need to right click our Domain and go to "New" and then select "Organizational Unit". Refer to image 47

![image](https://github.com/user-attachments/assets/d0a4f8fe-51c0-43db-abb0-ec9aef03e069)
Image 47

We are going to name it "IT" then click on "OK". The new organization will appear at the bottom of the list displayed from our Domain. Within this Organization we need to right click, select "New" and then "User". This new user is going to be called Karin Thornfield, her username is going to be KThorn and then click Next. As for the password we are going to type a super secure one. Finally click "Finish". Once the new user is created is going to appear in the IT organization. Refer to image 48.

![image](https://github.com/user-attachments/assets/518b72cd-3a89-4625-bb07-f06751d6a201)
Image 48


Next we are going to create another "Organization" called HR and then create another user.


3.2.- Configuring the Target Machine to join our Domain

To start configuring our Target Machine, we need to go to the search bar en search for "PC" and right click it and select "Properties". Inside the settings window, we can scroll down until we see an option called "Advanced System Settings". Then select the "Computer Name" tab and click "Change" and make sure to select "Domain". In the "Domain" section we are going to type the name of our Domain, the same we created earlier. Refer to image 49

![image](https://github.com/user-attachments/assets/ddc96bb0-f00a-4a80-a101-f7070bfb6382)
Image 49

An error window will be displayed because our Target Machine does not know how to resolve the Domain, so to fix it, we need to go to our Network Adapter in the bottom right corner of our screen, right click it and select "Open Network Internet settings". Once inside the network settings window, Select "Change adapter options", then right click our adapter, select "Properties" and then double click "Internet Protocol Version 4 (TCP/IPv4)". As we can see, previously we typed 8.8.8.8 as our Preferred DNS Server. We want to change it to the IP of our Windows Server. Refer to image 50

![image](https://github.com/user-attachments/assets/c935edc0-e8f6-436c-b001-2c18d3db6665)
Image 50

Now that we've changed the DNS Server IP, we can try to add the Target Machine again to a Domain. Use a Username and a Password to join the Domain we created. After you enter a Username and Password it will prompt you to restart.


After the restart, we now have an option to add a new user in the logging screen. Here we can use one of the User we created in the Server to login. Refer to image 51.

![image](https://github.com/user-attachments/assets/92cd7181-d34e-4316-a92d-cae80579fac8)
Image 51

After logging in with a User we have successfully added our Target Machine to our Domain.




### Step 4.- Generating Telemetry

4.1.- Preparing Kali Linux Virtual Machine

To generate Telemetry we are going to spin up a Kali Linux Virtual Machine.

First of all we are going to need to assign the IP we have in our diagram to our Kali VM. To do that we are going to right click the Network symbol in the top right corner of our screen and then select "Edit Connections...". Refer to image 52

![image](https://github.com/user-attachments/assets/82c62fe3-5a45-40be-9b29-07d3fc1c6ffe)
Image 52

Once inside we need to select our network adapter, then in the bottom left corner of the window we are going to select the cog button. Next we need to go to Ipv4 Settings. We need to change the "Method" from Automatic to manual so it can abilitate the option to add the IP address. After that click Add and finally type the IP address. At the end just click save. Refer to image 53

![image](https://github.com/user-attachments/assets/7f93f376-b326-4fa1-a2d0-058c95e60c25)
Image 53

For the changes to be applied, we need to click the network icon again and disconnect and connect.

Finally we need to update and upgrade our repository by typing "sudo apt-get update && sudo apt-get upgrade -y".



4.2.- Initiating the attack

First we need to create a new directory called AD-Project and to do that, we can use the command "mkdir AD-Project". All the files we will create and use will be saved into this directory.

For the attack we'll be using a tool called Crowbar. Crowbar (formally known as Levye) is a brute forcing tool that can be used during penetration tests. It was developed to brute force some protocols in a different manner according to other popular brute forcing tools. 

To install Crowbar we can head to the terminal and type the following command: "sudo apt-get install -y crowbar". After the installation is finished we are going to be using a Word List which is included in the kali linux machine called "RockYou" that can be found in /usr/share/wordlists. Refer to image 54.

![image](https://github.com/user-attachments/assets/4639b43a-ea22-47a3-85e0-8d6434160bd5)
Image 54

To use it we are going the RockYou file we need to unzip it with the command "sudo gunzip rockyou.txt.gz".

After unzipping the rockyou.txt file lets copy it and move it to our AD-Project directory we created earlier. we should be able to do that with the command "cp rockyou.txt ~/Desktop/AD-Project/".cd 

Going further we can check the size of the rockyou.txt file by using "ls -lh" command. As we can see is a rather heavy file, but for the sake of this project we are not going to be using the whole list. Refer to image 55.

![image](https://github.com/user-attachments/assets/8f748a0b-3183-4595-98ca-51eeb8e9aa76)
Image 55


To select only a part of the rockyou.txt file and then output it into another .txt file, we can use the command "head -n 20 rockyou.txt". The head command allows us to read the first lines of a file, with the -n flag we can specify how many lines to show of said file. refer to image 56.

![image](https://github.com/user-attachments/assets/d67b1e28-4331-41c5-8ea4-00a16b35a953)
Image 56

For the sake of the project we are going to edit in the password we set for the user we created in our server. We can do so by typing "nano passwords.txt" and typing it. To save the changes we can use CTRL + X, hit "Y" to save the changes and finally press Enter to exit the editor.


For the next step we want to enable Remote Desktop








































































































