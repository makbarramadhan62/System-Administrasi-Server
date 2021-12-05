# Report [Team 12] : Modul 2 - Automation

## Case Study

After created CodeIgniter framework with php5.6 and database server in MariaDB with phpMyAdmin. We will try to install 2 other frameworks needed, namely **Laravel** for landing pages and **WordPress** for blog. In this case we will install all of that using automation installation script with **Ansible **. Before that, Mr. Dzul as a senior programmer tell us about the ubuntu 18.04 Bionic is no longer supported for php7.4. Because of that, before we create the **Ansible **script, we will change the ubuntu of landing and blog to ubuntu 20.04 focal.

## Problem Solving

- **Laravel**

  - First, we need to change the ubuntu_landing from **bionic** to **focal** by typing this command `sudo lxc-create -n ubuntu_landing -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org`.

  - We can check the version of ubuntu by entering the LXC with `sudo lxc-attach -n ubuntu_landing` and type `lsb_release -a` to check the version of ubuntu.

    ![Laravel1](assets/Laravel1.PNG)

  - Because, we re-install the LXC of ubuntu_landing, then we need to do some configuration on ubuntu_landing. We can type `apt update; apt upgrade -y; apt install -y nano` to update and upgrade the ubuntu by **sources.list**. After that, we can set the **ip** from **dhcp** to static with same **ip** before we re-install this LXC by typing `nano /etc/netplan/10-lxc.yaml`. Save it, and type `netplan apply` to apply all the configurations we made.

  - We also need to configure the ssh server for this LXC. First, we need to install it by typing `apt install openssh-server` and then type `nano /etc/ssh/sshd_config` to doing some configurations below, then restart it by typing `service sshd restart`.

    ```
    PermitRootLogin yes
    RSAAuthentication yes
    ```

  - We can type `passwd` to create new password for our LXC, so we can access this LXC with ssh now. 

  - We can also set auto start on this LXC, using the same method as the module I described before. If you want to backup this LXC, first you need to stop your LXC by typing `sudo lxc-stop -n ubuntu_landing` and then type this command `lxc-copy -n ubuntu_landing -N ubuntu_landing_backup -sKD` to backup the LXC.

  - After we do all this LXC configurations, we can start making scripts **Ansible** for the **Laravel** installation.

  - First, we need to entering the modul2-ansible directory by typing `cd ~/ansible/modul2-ansible` and create **install-laravel.yml** file by typing `nano install-laravel.yml`.  In the **install-laravel.yml** file, we can type some configurations like this.

    ![Laravel2](assets/Laravel2.PNG)

  - As we can see in the **install-laravel.yml** configurations, the roles consist of 2 parts, namely **php** and **laravel**  .

  - First, we will create **php** role which contains the **php installation** and some **php configuration** by typing `mkdir -p role/php`. In the **php** directory, we need to create 2 directories, namely **tasks** and **handlers** by typing the command as below.

    ```
    mkdir -p roles/php/tasks
    mkdir -p roles/php/handlers
    ```

  - We need to go to the **tasks** directory in the **php** directory by typing `cd roles/php/tasks` and create **main.yml** file by typing `nano main.yml`. In the **main.yml** file we can type the php installation script as below. In case of **Laravel**, we need to install some additional extensions to php7.4 and some additional items for it to run properly.

    ```
    ---
    - name: delete apt chache
      become: yes
      become_user: root
      become_method: su
      command: rm -vf /var/lib/apt/lists/*
    
    - name: install php
      become: yes
      become_user: root
      become_method: su
      apt: name={{ item }} state=latest update_cache=true
      with_items:
        - gtkhash
        - crack-md5
        - git
        - curl
        - nginx
        - nginx-extras
        - php7.4
        - php7.4-fpm
        - php7.4-curl
        - php7.4-xml
        - php7.4-gd
        - php7.4-opcache
        - php7.4-mbstring
        - php7.4-zip
        - php7.4-json
        - php7.4-cli
    
    - name: enable module php mbstring
      command: phpenmod mbstring
      notify:
        - restart php
    ```

  - Still in the **php** directory, we move from the **tasks** directory to the **handlers** directory by typing `cd ../handlers` and create **main.yml** file by typing `nano main.yml`. In the **main.yml** file, we can type a script like the one below to restart php and nginx.

    ```
    ---
    - name: restart php
      become: yes
      become_user: root
      become_method: su
      action: service name=php7.4-fpm state=restarted
    
    - name: restart nginx
      become: yes
      become_user: root
      become_method: su
      action: service name=nginx state=restarted
    ```

  - Once we are done with the **php** role, we can move on to creating the **laravel** role. We need to create **laravel** role which contains the **Laravel installation** and some configuration by typing `mkdir -p role/laravel`. In the **laravel** directory, we need to create 3 directories, namely **tasks**, **handlers** and **templates** by typing the command as below.

    ```
    mkdir -p roles/laravel/tasks
    mkdir -p roles/laravel/handlers
    mkdir -p roles/laravel/templates
    ```

  - First, go to the **tasks** directory by typing `cd roles/laravel/tasks` and make **main.yml** file by typing `nano main.yml`. In the **main.yml** file, we can type a script like the one below to **install Laravel** and make some configurations for it run properly.

    ```
    ---
    - name: delete apt chache
      become: yes
      become_user: root
      become_method: su
      command: rm -vf /var/lib/apt/lists/*
    
    - name: Download and install Composer
      shell: curl -sS https://getcomposer.org/installer | php
      args:
        chdir: /usr/src/
        creates: /usr/local/bin/composer
        warn: false
      become: yes
    
    - name: Add Composer to global path
      copy:
        dest: /usr/local/bin/composer
        group: root
        mode: '0755'
        owner: root
        src: /usr/src/composer.phar
        remote_src: yes
      become: yes
    
    - name: Ansible delete file create-project
      file:
        path: /var/www/html/landing
        state: absent
    
    - name: composer create-project
      shell: /usr/local/bin/composer create-project laravel/laravel /var/www/html/landing --prefer-dist --no-interaction
    
    - name: Copy .env.template
      template:
        src=templates/env.template
        dest=/var/www/html/landing/.env
    
    - name: composer
      shell: cd /var/www/html/landing; /usr/local/bin/composer install  --no-interaction
    
    - name: key
      shell: /usr/bin/php7.4 /var/www/html/landing/artisan key:generate
    
    - name: chmod
      become: yes
      become_user: root
      become_method: su
      command: chmod 777 -R /var/www/html/landing/storage
    
    - name: Copy lv.conf
      template:
        src=templates/lv.conf
        dest=/etc/nginx/sites-available/{{ domain }}
      vars:
        servername: '{{ domain }}'
        
    - name: copy php7.conf
      template:
        src=templates/php7.conf
        dest=/etc/php/7.4/fpm/pool.d/www.conf
    
    - name: Symlink lv.conf
      command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
      notify:
        - restart nginx
    
    - name: Write {{ domain }} to /etc/hosts
      lineinfile:
        dest: /etc/hosts
        regexp: '.*{{ domain }}$'
        line: "127.0.0.1 {{ domain }}"
        state: present
    ```

  - We can move from **tasks** directory to **handlers** directory by typing `cd ../handlers` and create **main.yml** file by typing `nano main.yml`. In the **main.yml** file, we can type a script like the one below to restart the php and nginx.

    ```
    ---
    - name: restart php
      become: yes
      become_user: root
      become_method: su
      action: service name=php7.4-fpm state=restarted
    
    - name: restart nginx
      become: yes
      become_user: root
      become_method: su
      action: service name=nginx state=restarted
    ```

  - We can move from **handlers** directory to **templates** directory by typing `cd ../templates`. In the **templates** directory, we will make 2 files namely **lv.conf** and **env.template**.

  - First, we will create a **lv.conf** file by typing `nano lv.conf`. In the **lv.conf** file, we will create some configuration templates for nginx by typing the script as below.

    ```
    server {
         listen 80;
         listen [::]:80;
    
         # Log files for Debugging
         access_log /var/log/nginx/laravel-access.log;
         error_log /var/log/nginx/laravel-error.log;
    
         # Webroot Directory for Laravel project
         root /var/www/html/landing/public;
         index index.php index.html index.htm;
    
         # Your Domain Name
         server_name lxc_landing.dev;
    
         location / {
                 try_files $uri $uri/ /index.php?$query_string;
         }
    
         # PHP-FPM Configuration Nginx
         location ~ \.php$ {
                 try_files $uri =404;
                 fastcgi_split_path_info ^(.+\.php)(/.+)$;
                 fastcgi_pass 127.0.0.1:9001;
                 fastcgi_index index.php;
                 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                 include fastcgi_params;
         }
    }
    ```

  - After we created **lv.conf**, we will also create **env.template** file by typing `nano env.template`. In the **env.template** file, we will create a configuration template to connect **Laravel** to the **database server** by typing the script as below.

    ```
    APP_NAME=Landing
    APP_ENV=local
    APP_KEY=
    APP_DEBUG=true
    APP_URL=http://vm.local
    
    LOG_CHANNEL=stack
    LOG_DEPRECATIONS_CHANNEL=null
    LOG_LEVEL=debug
    
    DB_CONNECTION=mysql
    DB_HOST=10.0.3.200
    DB_PORT=3306
    DB_DATABASE=landing
    DB_USERNAME=admin
    DB_PASSWORD=admin
    
    BROADCAST_DRIVER=log
    CACHE_DRIVER=file
    FILESYSTEM_DRIVER=local
    QUEUE_CONNECTION=sync
    SESSION_DRIVER=file
    SESSION_LIFETIME=120
    
    MEMCACHED_HOST=127.0.0.1
    
    REDIS_HOST=127.0.0.1
    REDIS_PASSWORD=null
    REDIS_PORT=6379
    
    MAIL_MAILER=smtp
    MAIL_HOST=mailhog
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null
    MAIL_FROM_ADDRESS=null
    MAIL_FROM_NAME="${APP_NAME}"
    
    AWS_ACCESS_KEY_ID=
    AWS_SECRET_ACCESS_KEY=
    AWS_DEFAULT_REGION=us-east-1
    AWS_BUCKET=
    AWS_USE_PATH_STYLE_ENDPOINT=false
    
    PUSHER_APP_ID=
    PUSHER_APP_KEY=
    PUSHER_APP_SECRET=
    PUSHER_APP_CLUSTER=mt1
    
    MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
    MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
    ```

  - After we created **env.template**, we will also create **php7.conf** file by typing `nano php7.conf`. In the **php7.conf** file, we will create a configuration template to replace the configuration of **php7.4** from using **socket** to **port (127.0.0.1:9001)** by typing the script as below.

    ```
    ; Start a new pool named 'www'.
    ; the variable $pool can we used in any directive and will be replaced by the
    ; pool name ('www' here)
    [www]
    
    ; Per pool prefix
    ; It only applies on the following directives:
    ; - 'slowlog'
    ; - 'listen' (unixsocket)
    ; - 'chroot'
    ; - 'chdir'
    ; - 'php_values'
    ; - 'php_admin_values'
    ; When not set, the global prefix (or /usr) applies instead.
    ; Note: This directive can also be relative to the global prefix.
    ; Default Value: none
    ;prefix = /path/to/pools/$pool
    
    ; Unix user/group of processes
    ; Note: The user is mandatory. If the group is not set, the default user's group
    ;       will be used.
    user = www-data
    group = www-data
    
    ; The address on which to accept FastCGI requests.
    ; Valid syntaxes are:
    ;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific address on
    ;                            a specific port;
    ;   'port'                 - to listen on a TCP socket to all addresses on a
    ;                            specific port;
    ;   '/path/to/unix/socket' - to listen on a unix socket.
    ; Note: This value is mandatory.
    listen = 127.0.0.1:9001
    
    ; Set listen(2) backlog. A value of '-1' means unlimited.
    ; Default Value: 128 (-1 on FreeBSD and OpenBSD)
    ;listen.backlog = -1
    
    ; Set permissions for unix socket, if one is used. In Linux, read/write
    ; permissions must be set in order to allow connections from a web server. Many
    ; BSD-derived systems allow connections regardless of permissions.
    ; Default Values: user and group are set as the running user
    ;                 mode is set to 0666
    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0666
    
    ; List of ipv4 addresses of FastCGI clients which are allowed to connect.
    ; Equivalent to the FCGI_WEB_SERVER_ADDRS environment variable in the original
    ; PHP FCGI (5.2.2+). Makes sense only with a tcp listening socket. Each address
    ; must be separated by a comma. If this value is left blank, connections will be
    ; accepted from any ip address.
    ; Default Value: any
    ;listen.allowed_clients = 127.0.0.1
    
    ; Choose how the process manager will control the number of child processes.
    ; Possible Values:
    ;   static  - a fixed number (pm.max_children) of child processes;
    ;   dynamic - the number of child processes are set dynamically based on the
    ;             following directives. With this process management, there will be
    ;             always at least 1 children.
    ;             pm.max_children      - the maximum number of children that can
    ;                                    be alive at the same time.
    ;             pm.start_servers     - the number of children created on startup.
    ;             pm.min_spare_servers - the minimum number of children in 'idle'
    ;                                    state (waiting to process). If the number
    ;                                    of 'idle' processes is less than this
    ;                                    number then some children will be created.
    ;             pm.max_spare_servers - the maximum number of children in 'idle'
    ;                                    state (waiting to process). If the number
    ;                                    of 'idle' processes is greater than this
    ;                                    number then some children will be killed.
    ;  ondemand - no children are created at startup. Children will be forked when
    ;             new requests will connect. The following parameter are used:
    ;             pm.max_children           - the maximum number of children that
    ;                                         can be alive at the same time.
    ;             pm.process_idle_timeout   - The number of seconds after which
    ;                                         an idle process will be killed.
    ; Note: This value is mandatory.
    pm = dynamic
    
    ; The number of child processes to be created when pm is set to 'static' and the
    ; maximum number of child processes when pm is set to 'dynamic' or 'ondemand'.
    ; This value sets the limit on the number of simultaneous requests that will be
    ; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
    ; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
    ; CGI. The below defaults are based on a server without much resources. Don't
    ; forget to tweak pm.* to fit your needs.
    ; Note: Used when pm is set to 'static', 'dynamic' or 'ondemand'
    ; Note: This value is mandatory.
    pm.max_children = 5
    
    ; The number of child processes created on startup.
    ; Note: Used only when pm is set to 'dynamic'
    ; Default Value: min_spare_servers + (max_spare_servers - min_spare_servers) / 2
    pm.start_servers = 2
    
    ; The desired minimum number of idle server processes.
    ; Note: Used only when pm is set to 'dynamic'
    ; Note: Mandatory when pm is set to 'dynamic'
    pm.min_spare_servers = 1
    
    ; The desired maximum number of idle server processes.
    ; Note: Used only when pm is set to 'dynamic'
    ; Note: Mandatory when pm is set to 'dynamic'
    pm.max_spare_servers = 3
    
    ; The number of seconds after which an idle process will be killed.
    ; Note: Used only when pm is set to 'ondemand'
    ; Default Value: 10s
    ;pm.process_idle_timeout = 10s;
    
    ; The number of requests each child process should execute before respawning.
    ; This can be useful to work around memory leaks in 3rd party libraries. For
    ; endless request processing specify '0'. Equivalent to PHP_FCGI_MAX_REQUESTS.
    ; Default Value: 0
    pm.max_requests = 100
    
    ; The URI to view the FPM status page. If this value is not set, no URI will be
    ; recognized as a status page. It shows the following informations:
    ;   pool                 - the name of the pool;
    ;   process manager      - static, dynamic or ondemand;
    ;   start time           - the date and time FPM has started;
    ;   start since          - number of seconds since FPM has started;
    ;   accepted conn        - the number of request accepted by the pool;
    ;   listen queue         - the number of request in the queue of pending
    ;                          connections (see backlog in listen(2));
    ;   max listen queue     - the maximum number of requests in the queue
    ;                          of pending connections since FPM has started;
    ;   listen queue len     - the size of the socket queue of pending connections;
    ;   idle processes       - the number of idle processes;
    ;   active processes     - the number of active processes;
    ;   total processes      - the number of idle + active processes;
    ;   max active processes - the maximum number of active processes since FPM
    ;                          has started;
    ;   max children reached - number of times, the process limit has been reached,
    ;                          when pm tries to start more children (works only for
    ;                          pm 'dynamic' and 'ondemand');
    ; Value are updated in real time.
    ; Example output:
    ;   pool:                 www
    ;   process manager:      static
    ;   start time:           01/Jul/2011:17:53:49 +0200
    ;   start since:          62636
    ;   accepted conn:        190460
    ;   listen queue:         0
    ;   max listen queue:     1
    ;   listen queue len:     42
    ;   idle processes:       4
    ;   active processes:     11
    ;   total processes:      15
    ;   max active processes: 12
    ;   max children reached: 0
    ;
    ; By default the status page output is formatted as text/plain. Passing either
    ; 'html', 'xml' or 'json' in the query string will return the corresponding
    ; output syntax. Example:
    ;   http://www.foo.bar/status
    ;   http://www.foo.bar/status?json
    ;   http://www.foo.bar/status?html
    ;   http://www.foo.bar/status?xml
    ;
    ; By default the status page only outputs short status. Passing 'full' in the
    ; query string will also return status for each pool process.
    ; Example:
    ;   http://www.foo.bar/status?full
    ;   http://www.foo.bar/status?json&full
    ;   http://www.foo.bar/status?html&full
    ;   http://www.foo.bar/status?xml&full
    ; The Full status returns for each process:
    ;   pid                  - the PID of the process;
    ;   state                - the state of the process (Idle, Running, ...);
    ;   start time           - the date and time the process has started;
    ;   start since          - the number of seconds since the process has started;
    ;   requests             - the number of requests the process has served;
    ;   request duration     - the duration in µs of the requests;
    ;   request method       - the request method (GET, POST, ...);
    ;   request URI          - the request URI with the query string;
    ;   content length       - the content length of the request (only with POST);
    ;   user                 - the user (PHP_AUTH_USER) (or '-' if not set);
    ;   script               - the main script called (or '-' if not set);
    ;   last request cpu     - the %cpu the last request consumed
    ;                          it's always 0 if the process is not in Idle state
    ;                          because CPU calculation is done when the request
    ;                          processing has terminated;
    ;   last request memory  - the max amount of memory the last request consumed
    ;                          it's always 0 if the process is not in Idle state
    ;                          because memory calculation is done when the request
    ;                          processing has terminated;
    ; If the process is in Idle state, then informations are related to the
    ; last request the process has served. Otherwise informations are related to
    ; the current request being served.
    ; Example output:
    ;   ************************
    ;   pid:                  31330
    ;   state:                Running
    ;   start time:           01/Jul/2011:17:53:49 +0200
    ;   start since:          63087
    ;   requests:             12808
    ;   request duration:     1250261
    ;   request method:       GET
    ;   request URI:          /test_mem.php?N=10000
    ;   content length:       0
    ;   user:                 -
    ;   script:               /home/fat/web/docs/php/test_mem.php
    ;   last request cpu:     0.00
    ;   last request memory:  0
    ;
    ; Note: There is a real-time FPM status monitoring sample web page available
    ;       It's available in: ${prefix}/share/fpm/status.html
    ;
    ; Note: The value must start with a leading slash (/). The value can be
    ;       anything, but it may not be a good idea to use the .php extension or it
    ;       may conflict with a real PHP file.
    ; Default Value: not set
    pm.status_path = /php-status
    
    ; The ping URI to call the monitoring page of FPM. If this value is not set, no
    ; URI will be recognized as a ping page. This could be used to test from outside
    ; that FPM is alive and responding, or to
    ; - create a graph of FPM availability (rrd or such);
    ; - remove a server from a group if it is not responding (load balancing);
    ; - trigger alerts for the operating team (24/7).
    ; Note: The value must start with a leading slash (/). The value can be
    ;       anything, but it may not be a good idea to use the .php extension or it
    ;       may conflict with a real PHP file.
    ; Default Value: not set
    ;ping.path = /ping
    
    ; This directive may be used to customize the response of a ping request. The
    ; response is formatted as text/plain with a 200 response code.
    ; Default Value: pong
    ;ping.response = pong
    
    ; The access log file
    ; Default: not set
    ;access.log = log/$pool.access.log
    
    ; The access log format.
    ; The following syntax is allowed
    ;  %%: the '%' character
    ;  %C: %CPU used by the request
    ;      it can accept the following format:
    ;      - %{user}C for user CPU only
    ;      - %{system}C for system CPU only
    ;      - %{total}C  for user + system CPU (default)
    ;  %d: time taken to serve the request
    ;      it can accept the following format:
    ;      - %{seconds}d (default)
    ;      - %{miliseconds}d
    ;      - %{mili}d
    ;      - %{microseconds}d
    ;      - %{micro}d
    ;  %e: an environment variable (same as $_ENV or $_SERVER)
    ;      it must be associated with embraces to specify the name of the env
    ;      variable. Some exemples:
    ;      - server specifics like: %{REQUEST_METHOD}e or %{SERVER_PROTOCOL}e
    ;      - HTTP headers like: %{HTTP_HOST}e or %{HTTP_USER_AGENT}e
    ;  %f: script filename
    ;  %l: content-length of the request (for POST request only)
    ;  %m: request method
    ;  %M: peak of memory allocated by PHP
    ;      it can accept the following format:
    ;      - %{bytes}M (default)
    ;      - %{kilobytes}M
    ;      - %{kilo}M
    ;      - %{megabytes}M
    ;      - %{mega}M
    ;  %n: pool name
    ;  %o: ouput header
    ;      it must be associated with embraces to specify the name of the header:
    ;      - %{Content-Type}o
    ;      - %{X-Powered-By}o
    ;      - %{Transfert-Encoding}o
    ;      - ....
    ;  %p: PID of the child that serviced the request
    ;  %P: PID of the parent of the child that serviced the request
    ;  %q: the query string
    ;  %Q: the '?' character if query string exists
    ;  %r: the request URI (without the query string, see %q and %Q)
    ;  %R: remote IP address
    ;  %s: status (response code)
    ;  %t: server time the request was received
    ;      it can accept a strftime(3) format:
    ;      %d/%b/%Y:%H:%M:%S %z (default)
    ;  %T: time the log has been written (the request has finished)
    ;      it can accept a strftime(3) format:
    ;      %d/%b/%Y:%H:%M:%S %z (default)
    ;  %u: remote user
    ;
    ; Default: "%R - %u %t \"%m %r\" %s"
    ;access.format = %R - %u %t "%m %r%Q%q" %s %f %{mili}d %{kilo}M %C%%
    
    ; The log file for slow requests
    ; Default Value: not set
    ; Note: slowlog is mandatory if request_slowlog_timeout is set
    ;slowlog = log/$pool.log.slow
    
    ; The timeout for serving a single request after which a PHP backtrace will be
    ; dumped to the 'slowlog' file. A value of '0s' means 'off'.
    ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
    ; Default Value: 0
    ;request_slowlog_timeout = 0
    
    ; The timeout for serving a single request after which the worker process will
    ; be killed. This option should be used when the 'max_execution_time' ini option
    ; does not stop script execution for some reason. A value of '0' means 'off'.
    ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
    ; Default Value: 0
    ;request_terminate_timeout = 0
    
    ; Set open file descriptor rlimit.
    ; Default Value: system defined value
    ;rlimit_files = 1024
    
    ; Set max core size rlimit.
    ; Possible Values: 'unlimited' or an integer greater or equal to 0
    ; Default Value: system defined value
    ;rlimit_core = 0
    
    ; Chroot to this directory at the start. This value must be defined as an
    ; absolute path. When this value is not set, chroot is not used.
    ; Note: you can prefix with '$prefix' to chroot to the pool prefix or one
    ; of its subdirectories. If the pool prefix is not set, the global prefix
    ; will be used instead.
    ; Note: chrooting is a great security feature and should be used whenever
    ;       possible. However, all PHP paths will be relative to the chroot
    ;       (error_log, sessions.save_path, ...).
    ; Default Value: not set
    ;chroot =
    
    ; Chdir to this directory at the start.
    ; Note: relative path can be used.
    ; Default Value: current directory or / when chroot
    chdir = /
    
    ; Redirect worker stdout and stderr into main error log. If not set, stdout and
    ; stderr will be redirected to /dev/null according to FastCGI specs.
    ; Note: on highloaded environement, this can cause some delay in the page
    ; process time (several ms).
    ; Default Value: no
    catch_workers_output = yes
    
    ; Limits the extensions of the main script FPM will allow to parse. This can
    ; prevent configuration mistakes on the web server side. You should only limit
    ; FPM to .php extensions to prevent malicious users to use other extensions to
    ; exectute php code.
    ; Note: set an empty value to allow all extensions.
    ; Default Value: .php
    ;security.limit_extensions = .php .php3 .php4 .php5 .php7
    
    ; Pass environment variables like LD_LIBRARY_PATH. All $VARIABLEs are taken from
    ; the current environment.
    ; Default Value: clean env
    ;env[HOSTNAME] = $HOSTNAME
    env[PATH] = /srv/www/phpcs/scripts/:/usr/local/bin:/usr/bin:/bin
    ;env[TMP] = /tmp
    ;env[TMPDIR] = /tmp
    ;env[TEMP] = /tmp
    
    ; Additional php.ini defines, specific to this pool of workers. These settings
    ; overwrite the values previously defined in the php.ini. The directives are the
    ; same as the PHP SAPI:
    ;   php_value/php_flag             - you can set classic ini defines which can
    ;                                    be overwritten from PHP call 'ini_set'.
    ;   php_admin_value/php_admin_flag - these directives won't be overwritten by
    ;                                     PHP call 'ini_set'
    ; For php_*flag, valid values are on, off, 1, 0, true, false, yes or no.
    
    ; Defining 'extension' will load the corresponding shared extension from
    ; extension_dir. Defining 'disable_functions' or 'disable_classes' will not
    ; overwrite previously defined php.ini values, but will append the new value
    ; instead.
    
    ; Note: path INI options can be relative and will be expanded with the prefix
    ; (pool, global or /usr)
    
    ; Default Value: nothing is defined by default except the values in php.ini and
    ;                specified at startup with the -d argument
    ;php_admin_value[sendmail_path] = /usr/sbin/sendmail -t -i -f www@my.domain.com
    ;php_flag[display_errors] = off
    ;php_admin_value[error_log] = /var/log/fpm-php.www.log
    ;php_admin_flag[log_errors] = on
    ;php_admin_value[memory_limit] = 32M
    ```

  - Finally, after we created all configuration script of **Laravel** we can start to run the **Ansible** to make sure all script that we made was run properly with command `ansible-playbook -i hosts install-laravel.yml -k`.

    ![wp_laravel](assets/wp_laravel.PNG)

  - After the **Ansible** run properly, we can check in the browser by typing our domain which is `vm.local` to check **Laravel** was installed properly.

    ![laravel](assets/laravel.PNG)

- **WordPress**

  - First, we need to change the ubuntu_php7.4 from **bionic** to **focal** by typing this command `sudo lxc-create -n ubuntu_php7.4 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org`.
  
  - We can check the version of ubuntu by entering the LXC with `sudo lxc-attach -n ubuntu_php7.4` and type `lsb_release -a` to check the version of ubuntu.
  
    ![wordpress1](assets/wordpress1.PNG)
  
  - Because, we re-install the LXC of ubuntu_php7.4, then we need to do some configuration on ubuntu_php7.4. We can type `apt update; apt upgrade -y; apt install -y nano` to update and upgrade the ubuntu by **sources.list**. After that, we can set the **ip** from **dhcp** to static with same **ip** before we re-install this LXC by typing `nano /etc/netplan/10-lxc.yaml`. Save it, and type `netplan apply` to apply all the configurations we made.
  
  - We also need to configure the ssh server for this LXC. First, we need to install it by typing `apt install openssh-server` and then type `nano /etc/ssh/sshd_config` to doing some configurations below, then restart it by typing `service sshd restart`.
  
    ```
    PermitRootLogin yes
    RSAAuthentication yes
    ```
  
  - We can type `passwd` to create new password for our LXC, so we can access this LXC with ssh now. 
  
  - We can also set auto start on this LXC, using the same method as the module I described before. If you want to backup this LXC, first you need to stop your LXC by typing `sudo lxc-stop -n ubuntu_php7.4` and then type this command `lxc-copy -n ubuntu_php7.4 -N ubuntu_php7.4_backup -sKD` to backup the LXC.
  
  - After we do all this LXC configurations, we can start making scripts **Ansible **for the **WordPress** installation.
  
  - First, we need to entering the modul2-ansible directory by typing `cd ~/ansible/modul2-ansible` and create **install-wordpress.yml** file by typing `nano install-wordpress.yml`.  In the **install-wordpress.yml** file, we can type some configurations like this.
  
    ![wordpress2](assets/wordpress2.PNG)
  
  - As we can see in the **install-wordpress.yml** configurations, the roles consist of 1 parts, namely **WordPress**.
  
  - First, we will create **WordPress** role which contains the **php installation**, **php configuration**, **WordPress installation**, and **WordPress configuration** by typing `mkdir -p role/wordpress`. In the **wordpress** directory, we need to create 3 directories, namely **tasks**, **handlers**, and **templates** by typing the command as below.
  
    ```
    mkdir -p roles/wordpress/tasks
    mkdir -p roles/wordpress/handlers
    mkdir -p roles/wordpress/templates
    ```
  
  - First, go to the **tasks** directory by typing `cd roles/wordpress/tasks` and make **main.yml** file by typing `nano main.yml`. In the **main.yml** file, we can type a script like the one below to **install php**, **php configuration**, **install WordPress**, and **WordPress configuration**.
  
    ```
    ---
    - name: delete apt chache
      become: yes
      become_user: root
      become_method: su
      command: rm -vf /var/lib/apt/lists/*
    
    - name: install requirement
      become: yes
      become_user: root
      become_method: su
      apt: name={{ item }} state=latest update_cache=true
      with_items:
        - nginx
        - nginx-extras
        - curl
        - wget
        - php7.4
        - php7.4-fpm
        - php7.4-curl
        - php7.4-xml
        - php7.4-gd
        - php7.4-opcache
        - php7.4-mbstring
        - php7.4-zip
        - php7.4-json
        - php7.4-cli
        - php7.4-mysqlnd
        - php7.4-xmlrpc
        - php7.4-curl
    
    - name: wget wordpress
      shell: wget -c http://wordpress.org/latest.tar.gz
    
    - name: tar latest.tar.gz
      shell: tar -xvzf latest.tar.gz
    
    - name: copy folder wordpress
      shell: cp -R wordpress /var/www/html/blog
    
    - name: chmod
      become: yes
      become_user: root
      become_method: su
      command: chmod 775 -R /var/www/html/blog/
    
    - name: copy .wp-config.conf
      template:
        src=templates/wp.conf
        dest=/var/www/html/blog/wp-config.php
    
    - name: copy php7.conf
      template:
        src=templates/php7.conf
        dest=/etc/php/7.4/fpm/pool.d/www.conf
    
    - name: copy wordpress.conf
      template:
        src=templates/wordpress.conf
        dest=/etc/nginx/sites-available/{{ domain }}
      vars:
        servername: '{{ domain }}'
    
    - name: Symlink wordpress.conf
      command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
      notify:
        - restart nginx
    
    - name: Write {{ domain }} to /etc/hosts
      lineinfile:
        dest: /etc/hosts
        regexp: '.*{{ domain }}$'
        line: "127.0.0.1 {{ domain }}"
        state: present
    
    - name: enable module php mbstring
      command: phpenmod mbstring
      notify:
        - restart php
    ```
  
  - We can move from **tasks** directory to **handlers** directory by typing `cd ../handlers` and create **main.yml** file by typing `nano main.yml`. In the **main.yml** file, we can type a script like the one below to restart the php and nginx.
  
    ```
    ---
    - name: restart php
      become: yes
      become_user: root
      become_method: su
      action: service name=php7.4-fpm state=restarted
    
    - name: restart nginx
      become: yes
      become_user: root
      become_method: su
      action: service name=nginx state=restarted
    ```
  
  - We can move from **handlers** directory to **templates** directory by typing `cd ../templates`. In the **templates** directory, we will make 2 files namely **wp.conf** and **wordpress.conf**.
  
  - First, we will create a **wordpress.conf** file by typing `nano wordpress.conf`. In the **wordpress.conf** file, we will create some configuration templates for nginx by typing the script as below.
  
    ```
    server {
         listen 80;
         listen [::]:80;
    
         # Log files for Debugging
         access_log /var/log/nginx/wordpress-access.log;
         error_log /var/log/nginx/wordpress-error.log;
    
         # Webroot Directory for wordpress project
         root /var/www/html/blog;
         index index.php index.html index.htm;
    
         # Your Domain Name
         server_name lxc_php7.dev;
    
         location / {
                 try_files $uri $uri/ /index.php?$query_string;
         }
    
         # PHP-FPM Configuration Nginx
         location ~ \.php$ {
                 try_files $uri =404;
                 fastcgi_split_path_info ^(.+\.php)(/.+)$;
                 fastcgi_pass 127.0.0.1:9001;
                 fastcgi_index index.php;
                 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                 include fastcgi_params;
         }
    }
    ```
  
  - After we created **wordpress.conf**, we will also create **wp.conf** file by typing `nano wp.conf`. In the **wp.conf** file, we will create a configuration template to connect **WordPress** to the **database server** by typing the script as below.
  
    ```
    <?php
    /**
     * The base configuration for WordPress
     *
     * The wp-config.php creation script uses this file during the installation.
     * You don't have to use the web site, you can copy this file to "wp-config.php"
     * and fill in the values.
     *
     * This file contains the following configurations:
     *
     * * MySQL settings
     * * Secret keys
     * * Database table prefix
     * * ABSPATH
     *
     * @link https://wordpress.org/support/article/editing-wp-config-php/
     *
     * @package WordPress
     */
    
    define( 'WP_HOME', 'http://vm.local/blog' );
    define( 'WP_SITEURL', 'http://vm.local/blog' );
    
    // ** MySQL settings - You can get this info from your web host ** //
    /** The name of the database for WordPress */
    define( 'DB_NAME', 'blog' );
    
    /** MySQL database username */
    define( 'DB_USER', 'admin' );
    
    /** MySQL database password */
    define( 'DB_PASSWORD', 'admin' );
    
    /** MySQL hostname */
    define( 'DB_HOST', '10.0.3.200:3306' );
    
    /** Database charset to use in creating database tables. */
    define( 'DB_CHARSET', 'utf8' );
    
    /** The database collate type. Don't change this if in doubt. */
    define( 'DB_COLLATE', '' );
    
    /**#@+
     * Authentication unique keys and salts.
     *
     * Change these to different unique phrases! You can generate these using
     * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
     *
     * You can change these at any point in time to invalidate all existing cookies.
     * This will force all users to have to log in again.
     *
     * @since 2.6.0
     */
    define( 'AUTH_KEY',         'put your unique phrase here' );
    define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
    define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
    define( 'NONCE_KEY',        'put your unique phrase here' );
    define( 'AUTH_SALT',        'put your unique phrase here' );
    define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
    define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
    define( 'NONCE_SALT',       'put your unique phrase here' );
    
    /**#@-*/
    
    /**
     * WordPress database table prefix.
     *
     * You can have multiple installations in one database if you give each
     * a unique prefix. Only numbers, letters, and underscores please!
     */
    $table_prefix = 'wp_';
    
    /**
     * For developers: WordPress debugging mode.
     *
     * Change this to true to enable the display of notices during development.
     * It is strongly recommended that plugin and theme developers use WP_DEBUG
     * in their development environments.
     *
     * For information on other constants that can be used for debugging,
     * visit the documentation.
     *
     * @link https://wordpress.org/support/article/debugging-in-wordpress/
     */
    define( 'WP_DEBUG', false );
    
    /* Add any custom values between this line and the "stop editing" line. */
    
    
    
    /* That's all, stop editing! Happy publishing. */
    
    /** Absolute path to the WordPress directory. */
    if ( ! defined( 'ABSPATH' ) ) {
            define( 'ABSPATH', __DIR__ . '/' );
    }
    
    /** Sets up WordPress vars and included files. */
    require_once ABSPATH . 'wp-settings.php';
    ```
  
  - After we created **wp.conf**, we will also create **php7.conf** file by typing `nano php7.conf`. In the **php7.conf** file, we will create a configuration template to replace the configuration of **php7.4** from using **socket** to **port (127.0.0.1:9001)** by typing the script as below.
  
    ```
    ; Start a new pool named 'www'.
    ; the variable $pool can we used in any directive and will be replaced by the
    ; pool name ('www' here)
    [www]
    
    ; Per pool prefix
    ; It only applies on the following directives:
    ; - 'slowlog'
    ; - 'listen' (unixsocket)
    ; - 'chroot'
    ; - 'chdir'
    ; - 'php_values'
    ; - 'php_admin_values'
    ; When not set, the global prefix (or /usr) applies instead.
    ; Note: This directive can also be relative to the global prefix.
    ; Default Value: none
    ;prefix = /path/to/pools/$pool
    
    ; Unix user/group of processes
    ; Note: The user is mandatory. If the group is not set, the default user's group
    ;       will be used.
    user = www-data
    group = www-data
    
    ; The address on which to accept FastCGI requests.
    ; Valid syntaxes are:
    ;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific address on
    ;                            a specific port;
    ;   'port'                 - to listen on a TCP socket to all addresses on a
    ;                            specific port;
    ;   '/path/to/unix/socket' - to listen on a unix socket.
    ; Note: This value is mandatory.
    listen = 127.0.0.1:9001
    
    ; Set listen(2) backlog. A value of '-1' means unlimited.
    ; Default Value: 128 (-1 on FreeBSD and OpenBSD)
    ;listen.backlog = -1
    
    ; Set permissions for unix socket, if one is used. In Linux, read/write
    ; permissions must be set in order to allow connections from a web server. Many
    ; BSD-derived systems allow connections regardless of permissions.
    ; Default Values: user and group are set as the running user
    ;                 mode is set to 0666
    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0666
    
    ; List of ipv4 addresses of FastCGI clients which are allowed to connect.
    ; Equivalent to the FCGI_WEB_SERVER_ADDRS environment variable in the original
    ; PHP FCGI (5.2.2+). Makes sense only with a tcp listening socket. Each address
    ; must be separated by a comma. If this value is left blank, connections will be
    ; accepted from any ip address.
    ; Default Value: any
    ;listen.allowed_clients = 127.0.0.1
    
    ; Choose how the process manager will control the number of child processes.
    ; Possible Values:
    ;   static  - a fixed number (pm.max_children) of child processes;
    ;   dynamic - the number of child processes are set dynamically based on the
    ;             following directives. With this process management, there will be
    ;             always at least 1 children.
    ;             pm.max_children      - the maximum number of children that can
    ;                                    be alive at the same time.
    ;             pm.start_servers     - the number of children created on startup.
    ;             pm.min_spare_servers - the minimum number of children in 'idle'
    ;                                    state (waiting to process). If the number
    ;                                    of 'idle' processes is less than this
    ;                                    number then some children will be created.
    ;             pm.max_spare_servers - the maximum number of children in 'idle'
    ;                                    state (waiting to process). If the number
    ;                                    of 'idle' processes is greater than this
    ;                                    number then some children will be killed.
    ;  ondemand - no children are created at startup. Children will be forked when
    ;             new requests will connect. The following parameter are used:
    ;             pm.max_children           - the maximum number of children that
    ;                                         can be alive at the same time.
    ;             pm.process_idle_timeout   - The number of seconds after which
    ;                                         an idle process will be killed.
    ; Note: This value is mandatory.
    pm = dynamic
    
    ; The number of child processes to be created when pm is set to 'static' and the
    ; maximum number of child processes when pm is set to 'dynamic' or 'ondemand'.
    ; This value sets the limit on the number of simultaneous requests that will be
    ; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
    ; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
    ; CGI. The below defaults are based on a server without much resources. Don't
    ; forget to tweak pm.* to fit your needs.
    ; Note: Used when pm is set to 'static', 'dynamic' or 'ondemand'
    ; Note: This value is mandatory.
    pm.max_children = 5
    
    ; The number of child processes created on startup.
    ; Note: Used only when pm is set to 'dynamic'
    ; Default Value: min_spare_servers + (max_spare_servers - min_spare_servers) / 2
    pm.start_servers = 2
    
    ; The desired minimum number of idle server processes.
    ; Note: Used only when pm is set to 'dynamic'
    ; Note: Mandatory when pm is set to 'dynamic'
    pm.min_spare_servers = 1
    
    ; The desired maximum number of idle server processes.
    ; Note: Used only when pm is set to 'dynamic'
    ; Note: Mandatory when pm is set to 'dynamic'
    pm.max_spare_servers = 3
    
    ; The number of seconds after which an idle process will be killed.
    ; Note: Used only when pm is set to 'ondemand'
    ; Default Value: 10s
    ;pm.process_idle_timeout = 10s;
    
    ; The number of requests each child process should execute before respawning.
    ; This can be useful to work around memory leaks in 3rd party libraries. For
    ; endless request processing specify '0'. Equivalent to PHP_FCGI_MAX_REQUESTS.
    ; Default Value: 0
    pm.max_requests = 100
    
    ; The URI to view the FPM status page. If this value is not set, no URI will be
    ; recognized as a status page. It shows the following informations:
    ;   pool                 - the name of the pool;
    ;   process manager      - static, dynamic or ondemand;
    ;   start time           - the date and time FPM has started;
    ;   start since          - number of seconds since FPM has started;
    ;   accepted conn        - the number of request accepted by the pool;
    ;   listen queue         - the number of request in the queue of pending
    ;                          connections (see backlog in listen(2));
    ;   max listen queue     - the maximum number of requests in the queue
    ;                          of pending connections since FPM has started;
    ;   listen queue len     - the size of the socket queue of pending connections;
    ;   idle processes       - the number of idle processes;
    ;   active processes     - the number of active processes;
    ;   total processes      - the number of idle + active processes;
    ;   max active processes - the maximum number of active processes since FPM
    ;                          has started;
    ;   max children reached - number of times, the process limit has been reached,
    ;                          when pm tries to start more children (works only for
    ;                          pm 'dynamic' and 'ondemand');
    ; Value are updated in real time.
    ; Example output:
    ;   pool:                 www
    ;   process manager:      static
    ;   start time:           01/Jul/2011:17:53:49 +0200
    ;   start since:          62636
    ;   accepted conn:        190460
    ;   listen queue:         0
    ;   max listen queue:     1
    ;   listen queue len:     42
    ;   idle processes:       4
    ;   active processes:     11
    ;   total processes:      15
    ;   max active processes: 12
    ;   max children reached: 0
    ;
    ; By default the status page output is formatted as text/plain. Passing either
    ; 'html', 'xml' or 'json' in the query string will return the corresponding
    ; output syntax. Example:
    ;   http://www.foo.bar/status
    ;   http://www.foo.bar/status?json
    ;   http://www.foo.bar/status?html
    ;   http://www.foo.bar/status?xml
    ;
    ; By default the status page only outputs short status. Passing 'full' in the
    ; query string will also return status for each pool process.
    ; Example:
    ;   http://www.foo.bar/status?full
    ;   http://www.foo.bar/status?json&full
    ;   http://www.foo.bar/status?html&full
    ;   http://www.foo.bar/status?xml&full
    ; The Full status returns for each process:
    ;   pid                  - the PID of the process;
    ;   state                - the state of the process (Idle, Running, ...);
    ;   start time           - the date and time the process has started;
    ;   start since          - the number of seconds since the process has started;
    ;   requests             - the number of requests the process has served;
    ;   request duration     - the duration in µs of the requests;
    ;   request method       - the request method (GET, POST, ...);
    ;   request URI          - the request URI with the query string;
    ;   content length       - the content length of the request (only with POST);
    ;   user                 - the user (PHP_AUTH_USER) (or '-' if not set);
    ;   script               - the main script called (or '-' if not set);
    ;   last request cpu     - the %cpu the last request consumed
    ;                          it's always 0 if the process is not in Idle state
    ;                          because CPU calculation is done when the request
    ;                          processing has terminated;
    ;   last request memory  - the max amount of memory the last request consumed
    ;                          it's always 0 if the process is not in Idle state
    ;                          because memory calculation is done when the request
    ;                          processing has terminated;
    ; If the process is in Idle state, then informations are related to the
    ; last request the process has served. Otherwise informations are related to
    ; the current request being served.
    ; Example output:
    ;   ************************
    ;   pid:                  31330
    ;   state:                Running
    ;   start time:           01/Jul/2011:17:53:49 +0200
    ;   start since:          63087
    ;   requests:             12808
    ;   request duration:     1250261
    ;   request method:       GET
    ;   request URI:          /test_mem.php?N=10000
    ;   content length:       0
    ;   user:                 -
    ;   script:               /home/fat/web/docs/php/test_mem.php
    ;   last request cpu:     0.00
    ;   last request memory:  0
    ;
    ; Note: There is a real-time FPM status monitoring sample web page available
    ;       It's available in: ${prefix}/share/fpm/status.html
    ;
    ; Note: The value must start with a leading slash (/). The value can be
    ;       anything, but it may not be a good idea to use the .php extension or it
    ;       may conflict with a real PHP file.
    ; Default Value: not set
    pm.status_path = /php-status
    
    ; The ping URI to call the monitoring page of FPM. If this value is not set, no
    ; URI will be recognized as a ping page. This could be used to test from outside
    ; that FPM is alive and responding, or to
    ; - create a graph of FPM availability (rrd or such);
    ; - remove a server from a group if it is not responding (load balancing);
    ; - trigger alerts for the operating team (24/7).
    ; Note: The value must start with a leading slash (/). The value can be
    ;       anything, but it may not be a good idea to use the .php extension or it
    ;       may conflict with a real PHP file.
    ; Default Value: not set
    ;ping.path = /ping
    
    ; This directive may be used to customize the response of a ping request. The
    ; response is formatted as text/plain with a 200 response code.
    ; Default Value: pong
    ;ping.response = pong
    
    ; The access log file
    ; Default: not set
    ;access.log = log/$pool.access.log
    
    ; The access log format.
    ; The following syntax is allowed
    ;  %%: the '%' character
    ;  %C: %CPU used by the request
    ;      it can accept the following format:
    ;      - %{user}C for user CPU only
    ;      - %{system}C for system CPU only
    ;      - %{total}C  for user + system CPU (default)
    ;  %d: time taken to serve the request
    ;      it can accept the following format:
    ;      - %{seconds}d (default)
    ;      - %{miliseconds}d
    ;      - %{mili}d
    ;      - %{microseconds}d
    ;      - %{micro}d
    ;  %e: an environment variable (same as $_ENV or $_SERVER)
    ;      it must be associated with embraces to specify the name of the env
    ;      variable. Some exemples:
    ;      - server specifics like: %{REQUEST_METHOD}e or %{SERVER_PROTOCOL}e
    ;      - HTTP headers like: %{HTTP_HOST}e or %{HTTP_USER_AGENT}e
    ;  %f: script filename
    ;  %l: content-length of the request (for POST request only)
    ;  %m: request method
    ;  %M: peak of memory allocated by PHP
    ;      it can accept the following format:
    ;      - %{bytes}M (default)
    ;      - %{kilobytes}M
    ;      - %{kilo}M
    ;      - %{megabytes}M
    ;      - %{mega}M
    ;  %n: pool name
    ;  %o: ouput header
    ;      it must be associated with embraces to specify the name of the header:
    ;      - %{Content-Type}o
    ;      - %{X-Powered-By}o
    ;      - %{Transfert-Encoding}o
    ;      - ....
    ;  %p: PID of the child that serviced the request
    ;  %P: PID of the parent of the child that serviced the request
    ;  %q: the query string
    ;  %Q: the '?' character if query string exists
    ;  %r: the request URI (without the query string, see %q and %Q)
    ;  %R: remote IP address
    ;  %s: status (response code)
    ;  %t: server time the request was received
    ;      it can accept a strftime(3) format:
    ;      %d/%b/%Y:%H:%M:%S %z (default)
    ;  %T: time the log has been written (the request has finished)
    ;      it can accept a strftime(3) format:
    ;      %d/%b/%Y:%H:%M:%S %z (default)
    ;  %u: remote user
    ;
    ; Default: "%R - %u %t \"%m %r\" %s"
    ;access.format = %R - %u %t "%m %r%Q%q" %s %f %{mili}d %{kilo}M %C%%
    
    ; The log file for slow requests
    ; Default Value: not set
    ; Note: slowlog is mandatory if request_slowlog_timeout is set
    ;slowlog = log/$pool.log.slow
    
    ; The timeout for serving a single request after which a PHP backtrace will be
    ; dumped to the 'slowlog' file. A value of '0s' means 'off'.
    ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
    ; Default Value: 0
    ;request_slowlog_timeout = 0
    
    ; The timeout for serving a single request after which the worker process will
    ; be killed. This option should be used when the 'max_execution_time' ini option
    ; does not stop script execution for some reason. A value of '0' means 'off'.
    ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
    ; Default Value: 0
    ;request_terminate_timeout = 0
    
    ; Set open file descriptor rlimit.
    ; Default Value: system defined value
    ;rlimit_files = 1024
    
    ; Set max core size rlimit.
    ; Possible Values: 'unlimited' or an integer greater or equal to 0
    ; Default Value: system defined value
    ;rlimit_core = 0
    
    ; Chroot to this directory at the start. This value must be defined as an
    ; absolute path. When this value is not set, chroot is not used.
    ; Note: you can prefix with '$prefix' to chroot to the pool prefix or one
    ; of its subdirectories. If the pool prefix is not set, the global prefix
    ; will be used instead.
    ; Note: chrooting is a great security feature and should be used whenever
    ;       possible. However, all PHP paths will be relative to the chroot
    ;       (error_log, sessions.save_path, ...).
    ; Default Value: not set
    ;chroot =
    
    ; Chdir to this directory at the start.
    ; Note: relative path can be used.
    ; Default Value: current directory or / when chroot
    chdir = /
    
    ; Redirect worker stdout and stderr into main error log. If not set, stdout and
    ; stderr will be redirected to /dev/null according to FastCGI specs.
    ; Note: on highloaded environement, this can cause some delay in the page
    ; process time (several ms).
    ; Default Value: no
    catch_workers_output = yes
    
    ; Limits the extensions of the main script FPM will allow to parse. This can
    ; prevent configuration mistakes on the web server side. You should only limit
    ; FPM to .php extensions to prevent malicious users to use other extensions to
    ; exectute php code.
    ; Note: set an empty value to allow all extensions.
    ; Default Value: .php
    ;security.limit_extensions = .php .php3 .php4 .php5 .php7
    
    ; Pass environment variables like LD_LIBRARY_PATH. All $VARIABLEs are taken from
    ; the current environment.
    ; Default Value: clean env
    ;env[HOSTNAME] = $HOSTNAME
    env[PATH] = /srv/www/phpcs/scripts/:/usr/local/bin:/usr/bin:/bin
    ;env[TMP] = /tmp
    ;env[TMPDIR] = /tmp
    ;env[TEMP] = /tmp
    
    ; Additional php.ini defines, specific to this pool of workers. These settings
    ; overwrite the values previously defined in the php.ini. The directives are the
    ; same as the PHP SAPI:
    ;   php_value/php_flag             - you can set classic ini defines which can
    ;                                    be overwritten from PHP call 'ini_set'.
    ;   php_admin_value/php_admin_flag - these directives won't be overwritten by
    ;                                     PHP call 'ini_set'
    ; For php_*flag, valid values are on, off, 1, 0, true, false, yes or no.
    
    ; Defining 'extension' will load the corresponding shared extension from
    ; extension_dir. Defining 'disable_functions' or 'disable_classes' will not
    ; overwrite previously defined php.ini values, but will append the new value
    ; instead.
    
    ; Note: path INI options can be relative and will be expanded with the prefix
    ; (pool, global or /usr)
    
    ; Default Value: nothing is defined by default except the values in php.ini and
    ;                specified at startup with the -d argument
    ;php_admin_value[sendmail_path] = /usr/sbin/sendmail -t -i -f www@my.domain.com
    ;php_flag[display_errors] = off
    ;php_admin_value[error_log] = /var/log/fpm-php.www.log
    ;php_admin_flag[log_errors] = on
    ;php_admin_value[memory_limit] = 32M
    ```
  
  - Finally, after we created all configuration script of **WordPress** we can start to run the **Ansible** to make sure all script that we made was run properly with command `ansible-playbook -i hosts install-wordpress.yml -k`.
  
    ![wp_ansible](assets/wp_ansible.PNG)
  
  - After the **WordPress** run properly, we can check in the browser by typing our domain which is `vm.local` to check **WordPress** was installed properly.
  
    ![Wordpress](assets/Wordpress.PNG)

## Created By Team 12 [IT - 02 - 02]
- Muhammad Akbar Ramadhan [1202190019]
- Wiranti Maharani [1202190030]
