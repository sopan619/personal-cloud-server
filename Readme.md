# **Making our own Cloud Storage**
## __We will make our own personal cloud storage solution using ubuntu server and [Nextcloud](https://nextcloud.com/).__

### **Why you might consider this path over more traditional options like Onedrive or Google Drive?**

Basically with the mentioned services and many others which offer cloud storage, they collect your data and use it to target you with advertisement. Apart from this, these data is stored in multiple location across the globe. Last but not the lease, all these options cost you money, if you run out of storage space or want to use some advance features.  

### Today we will learn how to make our own cloud storage option for free, and store all the data in our own system.
## **Things we will learn today**
+ How to install [Ubuntu Server]((https://ubuntu.com/download/server))
+ How to setup a [Virtual Machine](https://www.windowscentral.com/best-free-virtual-machine-software-windows-10-and-11)
+ How to make a VPN for our personal use, using [Zerotier](https://www.zerotier.com/) 
+ How to setup [Nextcloud](https://nextcloud.com/)

## **Step 1**
First lets get one thing out of the way. This guide is focusing on installing and running the server on a **Virtual Machine in Windows 11.** It is recommended to install the server on a dedicated machine for long term use.

I am writing this blog/guide for my own reference as well, I am a relatively new guy in this field, so I am also learning as I do these experiments. I am using this same setup for some time now, and it is certainly not perfect, but it is my own cloud storage solution, and it is working as I envisioned it, so no complaints.

### **Install Ubuntu Server**

You can Download the [Ubuntu Server LTS Edition](https://ubuntu.com/download/server) from the official website. Once you have the ISO, you can make a [bootable pendrive](https://www.javatpoint.com/rufus) from it using something like [Rufus](https://rufus.ie/en/). In my case I used a virtual machine on my Windows 11 laptop. You can make your own with [Virtual Box.](https://www.virtualbox.org/wiki/Downloads)

![Making a VM](<src/virtual box setup.png>)
Simply give your VM a name and select the ISO you have downloaded, in order to make a virtual machine.
In the next page, select the username and password for your server and make sure **Hostname** should not have any space in it.

![Username server](<src/vm user.png>)
Click on Next and in the next screen allocate some CPU cores and RAM according to your system specifications.
![VM Specs](<src/vm spec.png>)

Next, make a virtual hard disk for your system, here if you are gonna keep using it from a VM setup, then select an adequate amount of Hard drive space. Note, this is dynamically allocated, so as you keep filling the storage drive, the size of this virtual disk will increase till the pre selected value on this screen. 
![vm hdd](<src/vm storage.png>)

Click next and then Start your Virtual Machine, it will walk you though the ubuntu server installation. [Refer this video](https://www.youtube.com/watch?v=YtH9D2SqBqA) if you need help.

## **Step 2**
### **Install nextCloud**
So, at this point you should have Ubuntu server running on either a VM or on a physical system. 
>*Please note, this can be done on a regular edition Ubuntu OS or any other Linux OS or even on Windows.* 
Run this command in the terminal to update your server before moving with other installs.
```
sudo apt update && sudo apt upgrade -y
```
Once done, move with the next command, which will install [Nextcloud](https://nextcloud.com/) in our system.
```
sudo snap install nextcloud
```
Next, run this command to make an account on Nextcloud.
```
sudo nextcloud.manual-install netvn password
```
>Note, here ***netvn*** will be your username and ***password*** will be your password for logging into ***Nextcloud***. So change it here as per your liking. Password you can change later as well.

We will come back to do more setup with Nextcloud, for now let us set up ***Zerotier.***
## **Step 3**
### **Zerotier Setup** 
Zerotier is needed, since we want to use our nextcloud service from outside our Local network as well. So for remote access, we will make a VPN with Zerotier, and whenever our external network is connected with the VPN, we will be able to access our Nextcloud server remotely.
#### Make an Zerotier Account or Sign in with Google
![Zerotier home](<src/zerotier.jpeg>)

Once done, click on **Create a Network**, and copy the unique ***Network ID.***
![Net ID](<src/zt net id.jpeg>)

## **Step 4**
### **Install Zerotier**

Run the following command in the terminal of our Ubuntu Server,
```
curl -s https://install.zerotier.com | sudo bash
```
Next run this command, but note that instead of \<netword id>, you paste the **Unique Network ID** we created earlier.
```
sudo zerotier-cli join <network_id>
```

<img src="src/zt member.jpeg" alt="zt member auth" style="width:60px;" align="right"/>

It should say, Status 200 OK, means you have joined the network successfully. Now open the Zerotier network page and scroll down to Members section. 

Your server should show up there as a new member, just check the tick box to authorize the member. You have to do the same thing for every device which joins for the first time in this VPN. 

Now we have joined this VPN from our server. Let's finish the setup process of Nextcloud.

## **Step 5**
### **Setup Nextcloud**

Now let's add a trusted domain and https to our Nextcloud setup, so we can access the Web Interface from the device IP address.

Run these commands in the terminal,
```
sudo nextcloud.occ config:system:set trusted_domains 1 --value=*
```
```
sudo nextcloud.enable-https self-signed
```
Next, set the ubuntu firewall to allow these ports,
```
sudo ufw allow 80,443/tcp
```
We are done now! Let's check the dashboard on a browser now.

First check the IP for your server,
```
Hostname -I
```
Paste the IP in your browser tab, and it should open the Nextcloud dashboard, login with the ***Username*** and ***Password*** we created earlier and you should see the welcome page of Nextcloud.

![nextcloud home](<src/nextcloud home.jpeg>)

You can go to files and access and control all files and folders, but first we will check the disk space of our storage system. 
<img src="admin sett.jpeg" alt="zt member auth" style="width:150px;" align="right"/>

Click on the profile icon in the top and click on Administration settings.

Then scroll and click on System. Here you can see all details about the VM or your physical system, IP address, software details, system load, disk space, file count and many more.

Note, from here you can do all the settings for your Nextcloud instance, security, sharing, privacy, user management etc.

So, in the Disk section you will notice, not all the disk space is showing up. We will quickly fix that now.


Open up the Terminal and run the following commands,
```
sudo lvresize -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```
```
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```
Next we just need to mount the drive,
```
sudo snap connect nextcloud:removable-media
```

Now lets go back to the Dashboard and simply refresh the page once, you should see the full capacity of your drive showing up for Cloud Storage Use.

![full disk space](<src/nc disk space.jpeg>)


Note, in my case I am using a VM, where I selected the virtual disk space capacity as 60 GB.

Congratulations, you now have a personal cloud storage solution where only you will control the data and all the data will be stored in your home or VM. All which is left now is to access this server from outside internet.

We can very easily do that from our Android or iOS device by using Nextcloud and Zerotier app.

Once we connect to the Zerotier VPN, it acts as if we are connected to our server locally via one network. So we get full access, and upload speeds are also reliable in my case, both 4G and WiFi.

![android screen zt](<src/android.jpg>)

Simply open the Zerotier android app (you can download from Play Store), and add the Unique Network ID we got from before, Join the network.

Go to the Zerotier web page like before, and authorize your new device. That's it, done. Now open Nextcloud and put your server IP address to access all the files. 

> Note, every time you access the server from outside internet, you need to connect to the VPN.

Similarly you can do it for iOS, Windows and Linux as well. Enjoy your safe and secure Cloud Storage solution.


I will link some other useful links here, which helped me though this process for troubleshooting and other stuffs.

The main video I followed to do this, is also linked here, check it out and make your own Cloud Server :)

## ***Install Nextcloud and Remote Access***

<a href="https://www.youtube.com/watch?v=SH00ySqLaqg" target="_blank">
 <img src="src/netvn%20yt.jpeg" alt="Watch the video" width="540" border="3" />
</a>


## **Useful Links**

+ [How to SSH Into a VirtualBox Ubuntu Server](https://www.makeuseof.com/how-to-ssh-into-virtualbox-ubuntu/)

+ [Nextcloud & VirtualBox - Can't access initial login page](https://www.reddit.com/r/NextCloud/comments/fpfxqc/nextcloud_virtualbox_cant_access_initial_login/)

+ [How to Forward Ports to a Virtual Machine and Use It as a Server](https://www.howtogeek.com/122641/how-to-forward-ports-to-a-virtual-machine-and-use-it-as-a-server/)


## Thank you, if this post helps you, then I am happy :)