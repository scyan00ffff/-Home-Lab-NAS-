# -Rasberry-Pi-NAS-

This repo documents the process of setting up and hosting a piece of NAS software on a raspberry pi 2B. The aim of this project was to experiment with basic networking concepts while utilising an old device that was unused. It should allow me a shared storage space, accessible from multiple machines, in which to store useful files such as a folder of common ISO files that I use often on multiple physical machines when setting up other virtual machines. A secondary use of the NAS is to store the file structure for my Obsidian notes vault, allowing me to access and edit these with continuous changes from multiple devices, without having to pay a subscription fee. This is a begginer project and the hardware used is not very powerful. I am using this project to get an understanding of various concepts to then expand my homelab in the future with more powerful hardware. 

Equipment/Software used:
- Raspberry Pi 2B
- Peripherals for setup
- SD card flashed with Raspberry Pi OS Bookworm Lite
- Open Media Vault software

Network Diagram:

             ┌───────────┼───────────────────────────────┐
             │           │                               │
     ┌───────▼───────┐   │                       ┌───────▼────────┐
     │ Raspberry Pi  │   │                       │Laptop A (Windows)       
     │ 2B (NAS)      │   │                       │ (Wi-Fi/Eth)    │
     │  eth0 ────────┘   │                       └────────────────┘
     │  SMB/NFS/SSH      │
     │  Hostname: PiNAS    │                     ┌────────────────┐
     └───────┬───────────┘                       │ Laptop B (Ubuntu)     
             │                                   │ (Wi-Fi/Eth)    │
             │ USB 2.0                           └────────────────┘
     ┌───────▼────────┐
     │ External HDD/  │                       ┌──────────────────┐
     │ SSD (ext4/NTFS)│                       │ Other Devices    │
     └────────────────┘                       │ (Printer, NVR,   │
                                              │  IoT, consoles)  │
                                              └──────────────────┘


Initial Setup:

Flash Raspberry Pi OS Bookworm Lite to a micro SD card using the Raspberry Pi imager, ensuring that you enable SSH and set the default hostname of the device to "raspberrypi", then install the SD card into the Pi and power it on. 

Installing and Upgrading the OS:

As the OS is a Lite version this means it is headless and does not have GUI. The easiest way to setup the system will therefore be Via SSH. On a Linux machine this can be done by simply opening the terminal and typing "ssh raspberrypi.local" or if you know the IP address of the Pi, "ssh IPADDRESS". However for this setup I am using my windows machine so I will be using Putty in order to SSH into the Pi. 

Initially to ensure the Pi has booted correctly I ran "ping rasberrypi.local" to make sure i received a response. 

<img width="842" height="141" alt="image" src="https://github.com/user-attachments/assets/92aa73ef-97a6-4786-9fde-d829b6da98ca" />

I can now use putty to ssh into the Pi and begin the OS setup process. However, in my case putty would not connect using the hostname of the device so I had to access my router control panel (Type 192.168.1.1 into a browser) and look for the Pi to find it's IP address and connect using this. 

Before installing any additional software it is best practice to run "sudo apt update" and "sudo apt upgrade" to ensure the OS and all packages are up to date. 

Installing OpenMediaVault:

Before installing the software we need to run a pre-install script that will ensure our ethernet connection stays persistent during the setup process.
The script command is "wget -O - https://raw.githubusercontent.com/OpenMediaVault-Plugin-Developers/installScript/master/preinstall | sudo bash"

Once this script has finished running we need to reboot the Pi using "sudo reboot" 

The next command installs OpenMediaVault: "wget -O - https://raw.githubusercontent.com/OpenMediaVault-Plugin-Developers/installScript/master/install | sudo bash" 

I found that this process takes a long time, be prepared to wait a while. 

Once this process has finished running the Pi should automatically reboot, if connecting to it via SSH like I did, you will need to reconnect. If for some reason the pi does not reboot, run "sudo reboot".

Configuring the Web Interface:

With OpenMediaVault now up and running on the Pi we can connect to the web interface by navigating to "http://IPADDRESS" with "IPADDRESS" being the IP of your Pi. 

This takes us to a page asking for login credentials, the defaults are: Username: admin, Password: openmediavault

<img width="1142" height="990" alt="image" src="https://github.com/user-attachments/assets/5abb2bf1-caf4-43a5-9d24-c9f4ac6faa44" />

We now have access to a blank dashboard that we will eventually fill out with widgets.

<img width="1211" height="753" alt="image" src="https://github.com/user-attachments/assets/7e911a97-a945-4512-89e7-88f6268c7feb" />

The first step will be to change the default username and password to something secure to protect our system from attackers. This is done by clicking the symbol of the person in the top right corner and selecting "Change password". 

To populate the dashboard with widgets, again select the icon of the person and select "Dashboard". This will bring you to a menu with various checkboxes for widgets. Choosing which to enable is entirely personal preference, I went with this setup:

<img width="903" height="1169" alt="image" src="https://github.com/user-attachments/assets/fff23f03-f193-4609-9634-1e38634d5314" />

At this point it is time to mount the hard drive/storage device you will be using. I plugged a Seagate 2TB hard drive into my Pi and navigated to the drives tab on the left-hand side of the dashboard. However, my drive was not recognised so I decided to reboot the system, the startup process was extremely slow, however my drive was being recognised as I analysed the bootup logs. Back in the web interface though my drive was still not being recognised. I figured it could be a power issue as I was currently running; I upgraded the power supply and rebooted once again. 

This time I managed to gain access to the web interface once more, however the drives section was stuck in a permanent load cycle, running "systemd-analyze blame" told me that the bootup was not yet finished even though I have access to the web interface. The dashboard also tells me that my CPU usage is sitting around 85-99%, therefore it may be a hardware limitation issue. 

After rebooting the system multiple times I deduced that the reasons behind the slow boot times and unresponsive web interface were linked to the Seagate drive being underpowered and the Pi stuck in a cycle of trying to mount it. With this in mind i have removed the drive from the setup for now and instead installed a 32GB USB stick as a temporary solution. With this in place the system booted much faster and the drive was instantly recognised under the Storage > Disks tab on the dashboard. 

As best practice I reformatted the drive by clicking the eraser icon in the top left corner. 

Configuring the File System: 

1. Go to the File systems tab and click the plus icon to create new
2. Choose EXT4 for its lower system requirements
3. Select the drive you wish to create this file system on, in my case the USB stick
4. Click save, and a new file system will be created. 

Now that the file system is created, we need to virtually mount it to our NAS by selecting it under the "Mount" sub tab

The file system will now appear:

<img width="903" height="335" alt="image" src="https://github.com/user-attachments/assets/2ecb1a49-c06e-4954-a1e7-e58be1af7326" />

Creating a Shared Folder:

Now we have the file system set up we need to create a shared folder that can be accessed by multiple devices on the same network. 

Click on the shared folders tab, click the plus icon and fill out the relevant information like so:

<img width="908" height="564" alt="image" src="https://github.com/user-attachments/assets/c1eee585-862f-4483-94ce-b0679dfb3fd8" />

Enabling SAMBA/CIFS:

The shared folder we have just created will not actually be shared until you enable a protocol such as SMB. To do this:

1. Click services
2. Click SMB/CIFS
3. Click settings
4. Toggle Enabled checkbox to on
5. Save

Adding shared folder to SMB:

We need to add an SMB share for the shared folder to be shared. To do this:

1. Click Shares
2. Click Plus
3. Select your shared folder
4. Choose whether to make the share private or public
5. Choose whether you want the share to be read only or read/write

Creating a user:

1. Click Users
2. Click Users again
3. Click plus icon
4. Enter details for user, such as name, password etc.

Mounting the network drive on windows:

1. Open file explorer and go to "This PC"
2. Click the three dots and select "Map Network Drive"
3. Choose a Letter for the Drive
4. In the folder bar enter "\\YOURIP\YOURSHAREDFOLDERNAME"
5. A popup will ask you for the user credentials you just created for the new user
6. The shared drive will appear in file explorer

<img width="755" height="192" alt="image" src="https://github.com/user-attachments/assets/0737f6a0-155b-4943-8d96-ee539b549858" />

Mounting the network drive on Linux:

1. Open file explorer
2. Click other locations
3. In the server box enter the IP in the format "smb://IPADDRESS/SHARENAME" and click connect
4. Login using the user credentials we set up
5. The shared drive will now appear in the file explorer

<img width="886" height="543" alt="Pasted image" src="https://github.com/user-attachments/assets/f773c938-ce25-4332-b567-6b17c27fbf87" />

You can now populate the drive with whatever data you want from any device, and the changes will be made across all.

For my use case I uploaded my Obsidian Notes vault so that i can access it from my windows machine and my Linux machine.

The shared drive will also streamline my workflow as there is no longer a need to send files to yourself via email to access them on another device or upload them to google drive. Any file you need access to on both devices can simply be stored in here. 

To test this out I set the Vault folder location for Obsidian to a file in the shared file and edited some notes. The notes seamlessly updated on both devices. This PiNAS setup will save me a significant amount of money paying for a cloud subscription service for Obsidian.

See Next Steps file for further improvements I made to this setup.


<img width="347" height="60" alt="ascii-text-art" src="https://github.com/user-attachments/assets/31c85762-1168-4995-85eb-49cba946a5ba" />


My aim for this improvement is to be able to access the OMV NAS from anywhere on any network, not just my own. To do this requries the use of a VPN Tunnel such as Tailascale which I will be using. However in my specific case my NAS is set up on a WiFi 
network that i do not have control over as it is controlled by my housing provider. To overcome this issue I have purchased a GL-iNet SFT1200 router that will take an internet connection in and broadcast my own private network out. It also allows me 
full access to its admin control panel to set up services like port forwarding and firewalls. 

The new network diagram for my small home lab NAS is now: 

                       [ HOUSING INTERNET ]
                                │
                      (Shared Network WAN)
                                │
                      [ Wall Ethernet Outlet ]
                                │
                                ▼
                         ┌─────────────┐
                         │   ROUTER    │
                         │  (SFT1200)  │
                         │             │
                         └─────┬───────┘
                               │
           ┌──────────────────┴────────────────────┐
           │                   │                    │
     [Wi-Fi Signal]       [LAN Port 1]         [LAN Port 2]
           │                   │                    │
      Wireless Devices   ┌────────────┐        ┌─────────────┐
     (Phones, Tablets,   │ Raspberry  │        │ Windows     │
       Other PCs, etc.)  │   Pi NAS   │        │  Laptop     │
                         │ (OMV NAS)  │        │             │
                         └────────────┘        └─────────────┘
I connected all devices as shown above and rebooted my PiNAS and tried to connect to its dashboard that I configured previously. However this was not possible and the PiNAS was not showing up as a device in the admin panel of the router either. Running an nmap scan for active devices on the subnet also didn't return the Pi.
Troubleshooting this issue lead me to believe that OMV had created a static IP for the Pi when initally setting it up on the shared housing network, meaning that it would never show up under my new routers subnet as it was assigned permenantly to the previous network. To fix this I needed to re-enable 
DHCP to assign a new IP address in the routers subnet to the Pi, therefore making it accessible. 

To do this I ran the command `sudo omv-firstaid` which brings up a GUI element where you can configure networking and turn on DHCP. 

Once this was enabled the Pi assigned itself a new IP address in the new routers subnet which allowed me to see it in the routers admin panel as well as reconnect to the OMV admin panel and access my Shared folders. 

<img width="1292" height="498" alt="Screenshot 2025-08-30 230943" src="https://github.com/user-attachments/assets/f14aa95b-0440-4870-a135-9b800962237c" />

Installing and setting up Tailscale:

1. Create Tailscale account in a web browser 
2. Use curl to setup the repo for tailscale 
3. Update the system then install tailscale with `sudo apt get-install tailscale -y`
4. Start the Tailscale service with `systemctl --now tailscaled`
5. Authenticate to tailscale with `sudo tailscale up --ssh`. This will provide a login URL to open in a browser and sign in to your tailscale account to 
add the device to your network 
6. Check the tailscale IP, using `tailscale ip -4`, which should be in the format, `100.x.x.x`

The PiNAS's shared folders are now accesible from any device on the tailscale network. Add more devices by downloading tailscale on them and signing in. 

To actually access the shared folders on Windows and Linux we need to repeat the steps we did previously when setting up the shared folders but change the IP
address of the network folders to the new Tailscale IP address.

After this step I once again have access to my shared folders, however this time they are accessible by any of my devices that are connected to the tailscale network (tailnet), even when not on my local network.

<img width="1198" height="709" alt="Screenshot 2025-08-30 230753" src="https://github.com/user-attachments/assets/b5fce16a-cf55-4a7e-a971-bb519135c7c4" />

<img width="695" height="194" alt="Screenshot 2025-08-30 230813" src="https://github.com/user-attachments/assets/edd6447a-5fa6-464c-9763-64378484a763" />

Now that the NAS is accesible anywhere I want it to have an easier to remember name than the IP address. I enabled magicDNS in my tailscale account
and set the hostname of my device to an easy to remember one of my choosing. Now to access the dashboard you only have to type http://hostname into a browser.

Finally to make sure that, in the case of a power cycle event, my NAS is still accessible I made sure tailscale was set to run on boot by checking:
`systemctl status tailscaled` to check then `sudo systemctl enable tailscaled`. 

Setting up system monitoring:

I want to be able to monitor my system usage for the physical hardware and NAS remotely so I am setting up Netdata as a powerful monitor accessible via a 
MagicDNS URL. 

1. Install Netdata with  `bash <(curl -Ss https://my-netdata.io/kickstart.sh)`
2. Configure Netdata to run on boot and start it
3. Install OMV specific intergration with `sudo apt install netdata-plugin-openmediavault -y`
4. Enable the service both on startup and immediately

Now I can access the dashboard via its MagicDNS URL's remotely to check on the status of my PiNAS

<img width="2459" height="1280" alt="image" src="https://github.com/user-attachments/assets/3e709da7-fa01-45f9-bbf5-7b3bdb2673cf" />

Running these services about reaches the hardware limitations for the Pi 2B. My plan for it going forward is for it to act as a dedicated Obsidian vault host, allowing me to sync my notes to my two laptops and phone, free of charge or subscription. I have future plans to upgrade my home lab hardware to a mini PC to properly host OMV with a meaningfull amount of storage, alongside other proccses. 


   












