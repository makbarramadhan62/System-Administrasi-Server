# Midterm Exam (Windows Server)

##### **By Muhammad Akbar Ramadhan [1202190019] [IT-02-02]**

## Installation Windows Server 2022 on VirtualBox

- Download windows server 2022 ISO file from https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022.

- In this case, i will use VirtualBox to installing the windows server 2022. You can download VirtualBox from https://www.virtualbox.org/wiki/Downloads. 

- Open the VirtualBox, and click on the **new** button to create a new virtual machine. Give some **name** to your VM, now we are installing the server version so i named it "windows server". Select **Microsoft Windows** as OS type and **other Windows (64-bit)** as version.

  <img src="Assets/1vb_2.png" alt="1vb_2"/>

- In this case, I will assigned RAM as big as 8 GB because i will use the Desktop GUI. After that, create a Virtual Hard disk and choose **VHD** for the file Type. Select the **Dynamic Allocated** and stipulating 50 GB for the hard disk size. 

- Select created **VM** from the left panel of VirtualBox and click on the **Start** button. 

- Add the ISO file of windows server 2022 by select the folder icon to open the file explorer window of your respective OS and choose it. Finally, hit **start** button to move forward for the installation of windows server 2022. 

  <img src="Assets/1vb_5.png" alt="1vb_5"/>

- For the first installation setup, we need to enter our preferences. After that, click next to continue the setup and click install to start the installation process.

  <img src="Assets/2wsins_1.png" alt="2wsins_1"/>

- In this case i will choose the **standard evaluation (desktop experience)**, because i will manage this server with Graphical User Interface(GUI). After choose the operating system, click next to continue the installation process.

  <img src="Assets/2wsins_3.png" alt="2wsins_3"/>

- For the type of installation, i will choose **custom** because in this case we are not upgrading any previous version of windows server.

  <img src="Assets/2wsins_4.png" alt="2wsins_4"/>

- Select the available hard drive that we have created on VirtualBox. If you want to create partitions, you can select the **new** option. But, in this case i will let it by default for the hard drive size without partitions, then we can click next to let the server use the whole drive to install the OS.

  <img src="Assets/2wsins_5.png" alt="2wsins_5"/>

- After the installation is completed, we can type a password to secure a built-in administrator account. 

  <img src="Assets/2wsins_6.png" alt="2wsins_6"/>

- Finally, the installation process of windows server 2022 is done. We can check the version of this windows server by command `win + r` to open windows run and then type **winver**.

  <img src="Assets/2wsins_7.png" alt="2wsins_7"/>



## Installation Active Directory Domain Services & Promote Server to a Domain Controller

- Before we are trying to install the active directory domain services, i will set my network adapter on VirtualBox from NAT to Bridge Adapter

  <img src="Assets/3confip_1.png" alt="3confip_1"/>

- After we changed the network adapter, we need to set the ip of windows server to static. i will set my static ip to 192.168.1.19  and same ip for my DNS server.

  <img src="Assets/3confip_2.png" alt="3confip_2"/>

- After we changed the ip, open the **server manager** and click to **add roles and features** to install feature with roles that we need in this server.

  <img src="Assets/4insAD_1.png" alt="4insAD_1"/>

- After entering the wizard, you can just click next for tab **before you begin**, and select **role-based on feature-based installation** on the tab **installation type**. 

   <img src="Assets/4insAD_3.png" alt="4insAD_3"/>

- In the tab **server selection** select **select a server from the server pool** then click next. 

   <img src="Assets/4insAD_4.png" alt="4insAD_4"/>

- In the tab **server roles** we need to select roles **active directory domain services** by click the check box, and the pop up window will come out and click **add features**.

  <img src="Assets/4insAD_6.png" alt="4insAD_6"/>

- In the tab **features** and **AD DS** we just need to click next.

- in the tab **confirmation** we can recheck all of the installation selections. I recommend you to check box **restart the destination server automatically if required** that will be restart the server if there is features that need to restart all of the system before its working. Then, click install to start the installation process.

  <img src="Assets/4insAD_10.png" alt="4insAD_10"/>

- After the installation of active directory is done, we can immediately make this machine to be a **domain controller** by click link **promote this server to a domain controller** to start the installation.

  <img src="Assets/5psdc_1.png" alt="5psdc_1"/>

- We will redirected to **active directory domain services configuration wizard** for the configuration of promote server to a domain controller. In the tab **deployment configuration**, we need to select **add a new forest** and type our root domain name. In this case, i will use **vm.local** as my root domain name. After that, click next to continue the process.

   <img src="Assets/5psdc_2.png" alt="5psdc_2"/>

- In the tab **domain controller option** we will choose some configuration settings. For the **forest functional level** i choose **windows server 2016** because it was the highest option available at this time. For the **domain functional level** i choose **window server 2016** too. I just let the **specify domain controller** by default, and for the last we need to enter the password that we have registered on the windows server. After that, click next to continue the process.

  <img src="Assets/5psdc_3.png" alt="5psdc_3"/>

- In the tab **DNS option** just uncheck **create DNS delegation** and then click next.

- In the tab **additional options** we just need to click next, because the domain name will be automatically updated by default same as **root domain name** and then click next.

- In the tab **paths** i will leave the **database folder, log files folder and SYSVOL folder** by default and then click next.

  <img src="Assets/5psdc_6.png" alt="5psdc_6"/>

- In the tab **review options**, we can recheck all the configurations that have been done before installing it and then click next.

- In the tab **prerequisites check**, we can recheck again to make sure that all the configurations it properly correct before installing it. If we have make sure all the configurations are correct, just click install to start the installation process.

  <img src="Assets/5psdc_9.png" alt="5psdc_9"/>

- After the installation is done and the server was restarted properly. We can checked the active directory domain service was installed or not by open the **server manager**. Click tab tools in the menu bar and you can see there is the active directory in the dropdown menu.

  <img src="Assets/4insAD_11.png" alt="4insAD_11"/>

- we also can check the active directory by ping the domain on the **command prompt** and type `ping (nama domain)`.

  <img src="Assets/4insAD_12.png" alt="4insAD_12"/>

- We can check the **domain controller** was installed or not by enter the **active directory users and computers** in the tools on menu bar. After that, you can click dropdown vm.local (domain name) and you can see the directory **domain controllers** there.

  <img src="Assets/5psdc_11.png" alt="5psdc_11"/>



## Installation DNS Server

In my case, DNS server was automatically installed when i install the **active directory** with **domain controller**. We can check it by open the **server manager** and click to **add roles and features**. As we can see, the DNS server is already Installed. 

<img src="Assets/6DNS_info.png" alt="6DNS_info"/>

We can also check it in tools at menu bar, and there is sub menu DNS in dropdown menu.

<img src="Assets/8confDNS_1.png" alt="8confDNS_1"/>

If the DNS Server is not already installed in your case, you can install it with select **DNS server** and continue it same step like installing active directory. To make sure the DNS is installed properly, i will make some configuration to check it.

- Open the **DNS** in the **tools**, and you will see the **DNS manager** window.

- Now we can go to the **forward lookup zones** folder then go to the **vm.local** folder. Right click in  **vm.local** folder and select **new host(A or AAA)...** to create a new host.

  <img src="Assets/8confDNS_2.png" alt="8confDNS_2"/>

- After that, we just need to enter our **ip address** in my case, i use **192.168.1.19** as my ip address for DNS server then click **add host**.

  <img src="Assets/8confDNS_3.png" alt="8confDNS_3"/>

- Now we need to go to the **reverse lookup zones** to make a pointer. First we need to create a **new zone** by right click in the **reverse lookup zones**, select **new zone** and keep following the process. After we created a new zone, we can right click and select a **new pointer** then enter the required information.

  <img src="Assets/8confDNS_5.png" alt="8confDNS_5"/>

- After that, we need to restart the DNS server by click right in the server, select **all task** and then **restart**.

  <img src="Assets/8confDNS_6.png" alt="8confDNS_6"/>

- we check that the domain server by ping the domain name n the command prompt with command `ping (domain name)`. 

  <img src="Assets/8confDNS_7.png" alt="8confDNS_7"/>

- we can also check the domain with command `nslookup (domain name)`.

  <img src="Assets/8confDNS_8.png" alt="8confDNS_8"/>

- For the last, i will check this domain server by ping it in my computer client and so far its work properly.

  <img src="Assets/8confDNS_10.png" alt="8confDNS_10"/>



## Installation NET Framework 3.5

- open the **server manager** and click to **add roles and features**.

- After that, click next next every tab until **server roles** tab, because **NET Framework 3.5** installer is in the tab features.

- In **Features Tab**, we need to select **NET Framework 3.5 features** by hit the check box then click next.

  <img src="Assets/7NFins_1.png" alt="7NFins_1"/>

- In the tab **confirmation** we can add **specify an alternate source path**. Before we click it, we need to open **file manager** and find **sxs** folder in `D:/sources/sxs` and copy that directory.

  <img src="Assets/7NFins_2.png" alt="7NFins_2"/>

- After that, we can back in the server manager, click link **specify an alternate source path** and paste that directory in the path column then click **ok**.

  <img src="Assets/7NFins_4.png" alt="7NFins_4"/>

- Then click Install to start the installation process.

  <img src="Assets/7NFins_6.png" alt="7NFins_6"/>

- After the installation process is done, we can check it in the **registry editor** by command `win + r` to open windows run and type **regedit**  then enter. We can check it in `Computer/HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/NET Framework Setup/NDP/v3.5`.

  <img src="Assets/7NFins_7.png" alt="7NFins_7"/>

  