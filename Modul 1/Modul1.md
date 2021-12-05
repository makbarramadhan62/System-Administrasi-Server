# Report [Team 12] : Modul 1 - Virtualization

## Case Study

Create an basic virtualization as a company profile website based on domain strategy that has explain by CTO.

- vm.local
  - Containing Landing Page
- vm.local/blog
  - Containing Blog
- vm.local/app
  - Containing Enterprise App

###### Initial scheme

![skema awal](Assets\skema awal.png)



After meeting with programmer team and discussing about initial scheme that we made, we got the info from programmer team that the ubuntu 16.04 xenial has a new change that no longer has standard support since April 2021. After that, we discussed with programmer team about a new scheme that will be made and produce a decision that the new scheme that will be created is.

###### New scheme

![Skema Akhir](Assets\Skema Akhir.png)

| Server Name                    | IP         | Domain                 |
| ------------------------------ | ---------- | ---------------------- |
| LXC ubuntu server 18.04 php7.4 | 10.0.3.101 | http://lxc_php7.dev    |
| LXC debian server 9 php5.6     | 10.0.3.102 | http://lxc_php5.dev    |
| LXC ubuntu server 16.04        | 10.0.3.103 | http://lxc_landing.dev |

## Problem Solving

- #### Rename ubuntu_php5.6 and change the IP following the new schema

  - first, change the name of  ubuntu_php5.6 to ubuntu_landing using command `sudo lxc-copy -R -n old_name -N new_name`. This command will copying the old name and than change it to the new name. In our case, we will rename the ubuntu_php5.6 to ubuntu_landing.  To make sure the name of ubuntu_php5.6 has changed, we can check using the command `sudo lxc-ls -f`.

    ![1](Assets\1.PNG)

  - After that, we can start to running the container using command `sudo lxc-start -n ubuntu_landing ` and enter the container using command `sudo lxc-attach -n ubuntu_landing`.
  
  - If we was entering the ubuntu_landing, we need to type this command `nano /etc/network/interfaces` to entering the interfaces directory to setting the IP of ubuntu_landing from 10.0.3.102 to 10.0.3.103 and save it.
  
    ![1_1](Assets\1_1.PNG)
    
  - After changing the IP, we need to restart the system using command `systemctl restart networking.service` that will be restarting the network. In my case its doesn't work properly because when i checked the IP using command `ifconfig`, the IP is doesn'n changed. So, the solution is restart manually, so we have to exit from the container using command `exit`. After that, stop the container using command `sudo lxc-stop -n ubuntu_landing` and start it again using command `sudo lxc-start -n ubuntu_landing`. When the ubuntu_landing is running, we can check the IP using command `sudo lxc-ls -f` and see the IP has changed.



- #### Install LXC Debian 9 and set the name to ubuntu_php5.6

  - LXC Debian 9 use distro debian and release strech. so we can install it using command `sudo lxc-create -n debian_php5.6 -t download -- --dist debian --release stretch --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org` . Then  we can check is it already installed using command `sudo lxc-ls -f`.
  
  ![2](Assets\2.PNG)

- #### Setup debian_php5.6

  - Setting IP

    - First, make sure the debian_php5.6 is running and then enter the container. 
    - After entering the debian_php5.6, we need to install the net-tools curl using commad `apt install nano net-tools curl`.
    - If the installation was finished, we need to go to the interfaces directory using commad `nano /etc/network/interfaces`  to setting the IP on debian_php5.6 from dhcp to static using ip 10.0.3.102.

    ![3](Assets\3.PNG)

    - Save it and restart the network using command `systemctl restart networking.service`.
    - Check the IP using command  `ifconfig` to make sure that the IP has been changed.

    ![3_1](Assets\3_1.PNG)

  - Setup the nginx and set the domain to **http://lxc_pghp5.dev**.

    - first, we need to Install nginx using command `sudo apt install nginx nginx-extras`.

    - Go to the sites-available directory using `cd /etc/nginx/sites-available`.

    - Make an empty file with the name lxc_php5.6.dev using command `touch lxc_php5.6.dev`.

    - Edit lxc_php5.6.dev file with nano text editor using command `nano lxc_php5.6.dev` , name it the server name with lxc_php5.dev and save it.

      ![3_2](Assets\3_2.PNG)

    - Go to the directory ../sites-enabled using command `cd ../sites-enabled`.

    - Create lxc_php5.6 file link from sites-available directory in sites-enabled directory using command `ln -s /etc/nginx/sites-available/lxc_php5.6.dev`.

    - Recheck the script in lxc_php5.6.dev using command `nginx -t`.

    - Reload the nginx to make sure it running well using command `nginx -s reload`.

    -  Go to the hosts directory using command `nano /etc/hosts`.

    - Add the localhost ip 127.0.0.1 of lxc_php5.6 with the file name lxc_php5.dev. 

      ![3_3](Assets\3_3.PNG)

    - After that, go to the html directory using command `cd /var/www/html`.

    - Create a new directory and name it lxc_php5.6 using command `mkdir lxc_php5.6`.

    - Copy the default index.nginx-debian.html to the lxc_php5.6 file and name it index.html using command `cp index.nginx-debian.html lxc_php5.6/index.html`.

    - go to the lxc_php5.6 directory using command `cd lxc_php5.6`.

    - Edit index.html file with nano text editor using command `nano index.html` and add script `<h2>dari LXC debian 9 php 5.6</h2>` to find out the identity of debian_php5.6 when accessing the container in the browser and save it.

      ![3_4](Assets\3_4.PNG)

    - Check file index.html to find out that the file was connected properly using command `curl -i http://lxc_php5.dev`. 

      ![3_5](Assets\3_5.PNG)

- #### Setup ubuntu_landing

  - Setup the nginx and set the domain to **http://lxc_landing.dev**.

    - This step is same like steps in setup nginx in debian_php5.6, we just need apply some adjustment according to scheme.

    - First, we need to make sure that the nginx was installed properly in this container, then we using the command `sudo apt install nginx nginx-extras`.

    - After that we need to go to the sites-available directory using `cd /etc/nginx/sites-available`.

    - Make an empty file with the name lxc_php5.6.dev using command `touch lxc_php5.6.dev`.

    - Edit lxc_php5.6.dev file with nano text editor using command `nano lxc_php5.6.dev` , name it the server name with lxc_landing.dev and save it.

      ![4](Assets\4.PNG)

    - Go to the directory ../sites-enabled using command `cd ../sites-enabled`.

    - Create lxc_php5.6 file link from sites-available directory in sites-enabled directory using command `ln -s /etc/nginx/sites-available/lxc_php5.6.dev`.

    - Recheck the script in lxc_php5.6.dev using command `nginx -t`.

    - Reload the nginx to make sure it running well using command `nginx -s reload`.

    -  go to the hosts directory using command `nano /etc/hosts`.

    - Add the localhost ip 127.0.0.1 of lxc_php5.6 with the file name lxc_landing.dev. 

      ![4_1](Assets\4_1.PNG)

    - After that, go to the html directory using command `cd /var/www/html`.

    - Create a new directory and name it lxc_php5.6 using command `mkdir lxc_php5.6`.

    - Copy the default index.nginx-debian.html to the lxc_php5.6 file and name it index.html using command `cp index.nginx-debian.html lxc_php5.6/index.html`.

    - go to the lxc_php5.6 directory using command `cd lxc_php5.6`.

    - Edit index.html file with nano text editor using command `nano index.html` and add script `<h2>Ini LXC Landing</h2>` to find out the identity of ubuntu_landing when accessing the container in the browser and save it.

      ![4_2](Assets\4_2.PNG)
    
    - Check index.html file to find out that the file was connected properly using command `curl -i http://lxc_landing.dev`. 
    
      ![4_3](Assets\4_3.PNG)

- #### Setup auto start on lxc ubuntu_landing

  - First we need to enter the root account using command `sudo su`.

  - After that, we need to go to the ubuntu_landing directory using command `cd /var/lib/lxc/ubuntu_landing`.

  - Edit config file with nano text editor using command `nano config` .

    ![5](Assets\5.PNG)

  - In the config file, add script `lxc.start.auto = 1` to turn on auto start ubuntu_landing.

    ![5_!](Assets\5_!.PNG)

  - Restart ubuntu server using command `reboot`.  After that, we can use command `sudo lxc-ls -f` for checking the ubuntu_landing status that was automatically running when the ubuntu server is turn on. We can see that the "autostart" has changed from 0 to 1, that means the auto start has running properly in ubuntu_landing.

    ![5_2](Assets\5_2.PNG)

- #### Setup vm.local for proxy_pass

  - Edit hosts file using command `sudo nano /etc/hosts` and add script lxc container IP with file name each container base on the new schema.

  ![6](Assets\6.PNG)

  - Setup nginx
    
    - Install nginx and nginx extras using command `sudo apt install nginx nginx-extras`
    - Go to the sites-available directory using `cd /etc/nginx/sites-available`.
    - Make an empty file with the name lxc_php5.6.dev using command `sudo touch vm.local`.
    - Edit vm.local file with nano text editor using command `sudo nano vm.local` .
    
  - On vm.local we will specify the url of each lxc container to be accessible in the browser according to its function.

    - lxc_php5.dev will be used as application server using url http://vm.local/app 
    - lxc_php7.dev will be used as blog server using url http://vm.local/blog 
    - lxc_landing.dev will be used as proxy server and landing page using url http://vm.local/

    ![6_1](Assets\6_1.PNG)
    
    - Create lxc_php5.6 file link from sites-available directory in sites-enabled directory using command `ln -s /etc/nginx/sites-available/vm.local`.
    - Recheck the script in vm.local using command `nginx -t`.
    - Reload the nginx to make sure it running well using command `nginx -s reload`.

- #### Accessing URL in browser

  - Setting Hosts file in notepad

    - Open notepad and run as administrator.
    - Open and edit hosts file in directory C:\Windows\System32\drivers\etc\hosts.
    - Add IP vm.local 192.168.100.30 in notepad.

    ![a](Assets\a.PNG)

  - Accessing landing page in browser (chrome) with url http://vm.local

    ![7](Assets\7.PNG)

  - Accessing Blog Server in browser (chrome) with url http://vm.local/blog

    ![7_1](Assets\7_1.PNG)

  - Accessing Application Server in browser (chrome) with url http://vm.local/app

    ![7_2](Assets\7_2.PNG)





## Analysis

##### why for php5.6 need can't use ubuntu 16.04, and need to change the os to debian 9?

Base on the info that we get from Programmer Team and base on the data on github, if the ubuntu 16.04 has not get any standard support and just will be available on payment option by ubuntu extended security maintenance on april 2021 latest. Because of that, we and Programmer Team decide to change the operating system to Debian 9 because untill now Debian 9 can be access for free and still get supported until 2022. Other than that, Debian 9 release date is not to far after Ubuntu 16.04 release. Therefore, we can assume that Ubuntu 16.04 and Debian 9 is not have big different. (src: https://github.com/oerdnj/deb.sury.org/issues/1567).

##### Why we should use LXC virtualization on the our website scheme that will be developed?

Because the LXC (Linux Containers) is a lightweight virtualization system on the linux operating system and can be accessed for free.  The LXC system also provides frequently used command lines to cover routine management tasks, such as Create, Start, and Delete. The LXC system also allow us to be able to run multiple virtual units simultaneously on the same host.

##### What is a proxy server? why can we consider that vm.local as a proxy server?

Proxy server is a system that have a intermediary function for computer to access the internet. The vm.local can be considered as a proxy server because the vm.local is in the host or admin account and have a function as a network communication facility that will be intermediary between our server that we made with the internet.  Therefore, the server that we made can be accessible on internet using browser.



## Created By Team 12 [IT - 02 - 02]

- Muhammad Akbar Ramadhan [1202190019]
- Wiranti Maharani [1202190030]