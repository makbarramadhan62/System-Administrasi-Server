# Ujian Tengah Semester  (Windows Server)

##### **By Muhammad Akbar Ramadhan [1202190019] [IT-02-02]**

## Installation Windows Server 2022 on VirtualBox

- Download windows server 2022 ISO file from https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022.

- In this case, we will use VirtualBox to installing the windows server 2022. You can download VirtualBox from https://www.virtualbox.org/wiki/Downloads. 

- Open the VirtualBox, and click on the **new** button to create a new virtual machine. Give some **name** to your VM, here we are installing the server version so i named it "windows server". Select **Microsoft Windows** as OS type and **other Windows (64-bit)** as version.

  <img src="aset foto/1vb_2.png" alt="1vb_2"/>

- In this case, i will assigned RAM as big as 8 GB because i will use the Desktop GUI. After that, i will create a Virtual Hard disk and choose **VHD** for the file Type. Select the **Dynamic Allocated** and stipulating 50 GB for the hard disk size. 

- Select the created **VM** from the left panel of VirtualBox and click on the **Start** button given in the Mene. After that, add the ISO file of windows server 2022 by select the folder icon to open the file explorer window of your respective OS and choose it. Finally, hit **start** button to move forward for the installation of windows server 2022. 

  ![1vb_5](aset foto/1vb_5.png)

- For the first installation setup, we need to enter our preferences. After that, click next to continue the setup and click install to start the installation process.

  <img src="aset foto/2wsins_1.png" alt="2wsins_1"/>

- In this case i will choose the **standard evaluation (desktop experience)**, because i will manage this server with Graphical User Interface(GUI). After choose the operating system, click next to continue the installation process.

  ![2wsins_3](aset foto/2wsins_3.png)

- For the type of installation, i will choose **custom** because in this case we are not upgrading any previous version of windows server.

  ![2wsins_4](aset foto/2wsins_4.png)

- Select the available hard drive that we have created on VirtualBox. If you want to create partitions, you can select the **new** option. But, in this case i will let it by default for the hard drive size without partitions, then we can click next to let the server use the whole drive to install the OS.

  ![2wsins_5](aset foto/2wsins_5.png)

- After the installation is completed, we can type a password to secure a built-in administrator account. 

  ![2wsins_6](aset foto/2wsins_6.png)

- Finally, the installation process of windows server 2022 is done. We can check the version of this windows server by command `win + r` to open windows run and then type **winver**.

  ![2wsins_7](aset foto/2wsins_7.png)



## Installation Active Directory Domain Services & Promote Server to a Domain Controller

- Before we are trying to install the active directory domain services, i will set my network adapter on VirtualBox from NAT to Bridge Adapter

  ![3confip_1](aset foto/3confip_1.png)

- After we changed the network adapter, we need to set the ip of windows server to static. i will set my static ip to 192.168.1.19  and same ip for my DNS server.

  ![3confip_2](aset foto/3confip_2.png)

- After we changed the ip, open the **server manager** and click to **add roles and features** to install feature with roles that we need in this server.

  ![4insAD_1](aset foto/4insAD_1.png)

- After entering the wizard, you can just click next for tab **before you begin**, and select **role-based on feature-based installation** on the tab **installation type**. 

- In the tab **server selection** select **select a server from the server pool** then click next.  

- In the tab **server roles** we need to select roles **active directory domain services** by click the check box, and the pop up window will come out and click **add features**.

  ![4insAD_6](aset foto/4insAD_6.png)

- In the tab **features** and **AD DS** we just need to click next.

- in the tab **confirmation** we can recheck all of the installation selections. I recommend you to check box **restart the destination server automatically if required** that will be restart the server if there is features that need to restart all of the system before its working. Then, click install to start the installation process.

  ![4insAD_10](aset foto/4insAD_10.png)

- After the installation of active directory is done, we can immediately make this machine to be a **domain controller** by click link **promote this server to a domain controller** to start the installation.

  ![5psdc_1](aset foto/5psdc_1.png)

- We will redirected to **active directory domain services configuration wizard** for the configuration of promote server to a domain controller. In the tab **deployment configuration**, we need to select **add a new forest** and type our root domain name. In this case, i will use **vm.local** as my root domain name. After that, click next to continue the process.

   ![5psdc_2](aset foto/5psdc_2.png)

- In the tab **domain controller option** we will choose some configuration settings. For the **forest functional level** i choose **windows server 2016** because it was the highest option available at this time. For the **domain functional level** i choose **window server 2016** too. I just let the **specify domain controller** by default, and for the last we need to enter the password that we have registered on the windows server. After that, click next to continue the process.

  ![5psdc_3](aset foto/5psdc_3.png)

- In the tab **DNS option** just uncheck **create DNS delegation** and then click next.

- In the tab **additional options** we just need to click next, because the domain name will be automatically updated by default same as **root domain name** and then click next.

- In the tab **paths** i will leave the **database folder, log files folder and SYSVOL folder** by default and then click next.

  ![5psdc_6](aset foto/5psdc_6.png)

- In the tab **review options**, we can recheck all the configurations that have been done before installing it and then click next.

- In the tab **prerequisites check**, we can recheck again to make sure that all the configurations it properly correct before installing it. If we have make sure all the configurations are correct, just click install to start the installation process.

  ![5psdc_9](aset foto/5psdc_9.png)

- After the installation is done and the server was restarted properly. We can checked the active directory domain service was installed or not by open the **server manager**. Click tab tools in the menu bar and you can see there is the active directory in the dropdown menu.

  ![4insAD_11](aset foto/4insAD_11.png)

- we also can check the active directory by ping the domain on the **command prompt** and type `ping (nama domain)`.

  ![4insAD_12](aset foto/4insAD_12.png)

- We can check the **domain controller** was installed or not by enter the **active directory users and computers** in the tools on menu bar. After that, you can click dropdown vm.local (domain name) and you can see the directory **domain controllers** there.

  ![5psdc_11](aset foto/5psdc_11.png)



## Installation DNS Server

In my case, DNS server was automatically installed when i install the **active directory** with **domain controller**. We can check it by open the **server manager** and click to **add roles and features**. As we can see, the DNS server is already Installed. 

![6DNS_info](aset foto/6DNS_info.png)

We can also check it in tools at menu bar, and there is sub menu DNS in dropdown menu.

![8confDNS_1](aset foto/8confDNS_1.png)

If the DNS Server is not already installed in your case, you can install it with same step like installing active directory. You just need to select **DNS Server** in the tab **server roles**. To make sure the DNS is installed properly, i will make some configuration.

- Open the DNS, and you will see the DNS manager window.

- Now we can go to the **forward lookup zones** folder then go to the **vm.local** folder. Right click in  **vm.local** folder and select **new host(A or AAA)...** to create a new host.

  ![8confDNS_2](aset foto/8confDNS_2.png)

- After that, we just need to enter our **ip address** in my case, i use **192.168.1.19** as my ip address for DNS server then click **add host**.

  ![8confDNS_3](aset foto/8confDNS_3.png)

- Now we need to go to the **reverse lookup zones** to make a pointer. First we need to create a **new zone** by right click in the **reverse lookup zones**, select **new zone** and keep following the process. After we created a new zone, we can right click and select a **new pointer** then enter the required information.

  ![8confDNS_5](aset foto/8confDNS_5.png)

- After that, we need to restart the DNS server by click right in the server, select **all task** and then **restart**.

  ![8confDNS_6](aset foto/8confDNS_6.png)

- we check that the domain server by ping the domain name n the command prompt with command `ping (domain name)`. 

  ![8confDNS_7](aset foto/8confDNS_7.png)

- we can also check the domain with command `nslookup (domain name)`.

  ![8confDNS_8](aset foto/8confDNS_8.png)

- For the last, i will check this domain server by ping it in my computer client and so far its work properly.

  ![8confDNS_10](aset foto/8confDNS_10.png)



## Installation NET Framework 3.5

- open the **server manager** and click to **add roles and features**.

- After that, click next next every tab until tab server roles because **NET Framework 3.5** installer is in the tab features.

- In **Features Tab**, we need to select **NET Framework 3.5 features** by hit the check box then click next.

  ![7NFins_1](aset foto/7NFins_1.png)

- In the tab **confirmation** we can add **specify an alternate source path**. Before we click it, we need to open **file manager** and find **sxs** folder in `D:/sources/sxs` and copy that directory.

  ![7NFins_2](aset foto/7NFins_2.png)

- After that, we can back in the server manager, click link **specify an alternate source path** and paste that directory in the path column then click **ok**.

  ![7NFins_4](aset foto/7NFins_4.png)

- Then click Install to start the installation process.

  ![7NFins_6](aset foto/7NFins_6.png)

- After the installation process is done, we can check it in the **registry editor** by command `win + r` to open windows run and type **regedit**  then enter. We can check it in `Computer/HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/NET Framework Setup/NDP/v3.5`.

  ![7NFins_7](aset foto/7NFins_7.png)

  