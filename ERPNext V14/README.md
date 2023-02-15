# Guide-to-Install-Frappe-ERPNext-in-Ubuntu-22.04-LTS
A complete Guide to Install Frappe Bench in Ubuntu 22.04 LTS and install Frappe/ERPNext Application

### Pre-Requisites 

    Python 3.10+ (v14)
    Node.js 16
    Redis 6                                       (caching and realtime updates)
    MariaDB 10.6.x                                (to run database driven apps)
    yarn 1.12+                                    (js dependency manager)
    pip 22+                                       (py dependency manager)
    wkhtmltopdf (version 0.12.5 with patched qt)  (for pdf generation)
    cron                                          (bench's scheduled jobs: automated certificate renewal, scheduled backups)
    NGINX                                         (proxying multitenant sites in production)

### Update System Packages
It is always a good idea to upgrade the Ubuntu package if anything is available, run the below command to upgrade and update.
    
    sudo apt update && sudo apt upgrade -y

Check the server's current timezone
    
    date

You can run the following command to select your timezone.

    timedatectl list-timezones

Set correct timezone as per your region

    timedatectl set-timezone "Asia/Kolkata"
  
It is always recommended to reboot the server once the upgrade is done.

    sudo reboot

### Create a new user
I am using **erpnext** as frappe-user

#### you should not use _frappe_ or _erpnext_ as a frappe-user on your production server.

    sudo adduser erpnext
    sudo usermod -aG sudo erpnext
    su - erpnext  

### Install git
Git is the most commonly used version control system. Git tracks the changes you make to files, so you have a record of what has been done, and you can revert to specific versions should you ever need to. Git also makes collaboration easier, allowing changes by multiple people to all be merged into one source.
    
    sudo apt install git -y

### Install python-dev and python3-venv
python-dev is the package that contains the header files for the Python C API, 
which is used by lxml because it includes Python C extensions for high performance.

virtualenv is a tool for creating isolated Python environments containing their own copy of python , pip  and their own place to keep libraries installed from PyPI.
It's designed to allow you to work on multiple projects with different dependencies 
at the same time on the same machine.

    sudo apt install python3-dev python3-venv -y

### Install setuptools and pip (Python's Package Manager).
Setuptools is a collection of enhancements to the Python distutils that allow developers to more easily build and distribute Python packages, especially ones that have dependencies on other packages. Packages built and distributed using setuptools 
look to the user like ordinary Python packages based on the distutils.

Pip is a package manager for Python.  It's a tool that allows you to install and manage 
additional libraries and dependencies that are not distributed as part of the standard library.

    sudo apt install python3-setuptools python3-pip -y

### Install software-properties-common
Now install the below package to manage the repository, usually, Ubuntu 22.04 has already installed it, but for the safe side, we will run this command.

    sudo apt install software-properties-common -y
 if prompt for "Override local changes to /etc/pam.d/common-*?" on PAM Configuration, then safely choose "No".

### Install MariaDB 10.6 stable package
MariaDB is developed as open source software and as a relational database it provides an SQL interface for accessing data.
 
For ubuntu 22.04

    sudo apt install mariadb-server -y

IMPORTANT: During this installation you'll be prompted to set the MySQL root password.
If you are not prompted for the same You can initialize the MySQL server setup by executing the following command
    
    sudo mysql_secure_installation
    
#### Prompt
``` 
Enter current password for root (enter for none):   (safely press Enter)
Switch to unix_socket authentication [Y/n]          (Press "Y")
Change the root password? [Y/n]                     (Press "Y")
New password:                                       ("Enter new Password")
Re-enter new password:                              ("Re-enter new Password")
Remove anonymous users? [Y/n]                       (Press "Y")
Disallow root login remotely? [Y/n]                 (Press "Y") //If Press "N" then You want to access the database from a remote server for using business analytics software like Metabase / PowerBI / Tableau, etc.
Remove test database and access to it? [Y/n]        (Press "Y")
Reload privilege tables now? [Y/n]                  (Press "Y")
```

### MySQL database development files

    sudo apt install libmysqlclient-dev -y

### Edit the mariadb configuration (unicode character encoding)
You need to ensure to change the default character set of MySQL or MariaDB to Unicode instead of general. To do this you will need to edit the maria DB configuration file which is in this version located at /etc/mysql/mariadb.conf.d directory so you can directly edit this or locate the folder and then edit the file by typing the below command.

    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

Once the file opens you need to locate the line where collation-server says general and you need to modify it to Unicode as below,

    collation-server = utf8mb4_general_ci

Modify above as below.

    collation-server = utf8mb4_unicode_ci

Now press (Ctrl-X) and Save then exit.

And also locate my.cnf and edit the below configuration.

    sudo nano /etc/mysql/my.cnf

Make sure your configuration has the below lines in the file.

    [mysqld]
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci

    [mysql]
    default-character-set = utf8mb4

Now press (Ctrl-X) and Save then exit.

    sudo service mysql restart

### Install Redis
Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache, and message broker.
    
    sudo apt install redis-server -y
### Install Node.js 16 package
##### Ref: https://github.com/nvm-sh/nvm
    
    sudo apt install curl
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
    source ~/.profile
    nvm install v16

Now it has been installed you can now check the version by typing the below command.

    node -v

### Install Yarn using NPM
Now we will install Yarn which is a software packaging system developed by Facebook for Node.js, this is open source, so we will install it using npm.

    sudo apt install npm -y
    sudo npm install -g yarn

### Install wkhtmltopdf
Wkhtmltopdf is an open source simple and much effective command-line shell utility that enables user to convert any given HTML (Web Page) to PDF document or an image (jpg, png, etc)

    sudo apt install xvfb libfontconfig wkhtmltopdf -y
    
### Install NGINX
Nginx is an open-source web server that, since its initial success as a web server, 
is now also used as a reverse proxy, HTTP cache, and load balancer.

    sudo apt -y install nginx
    
### Install frappe-bench
The bench is a Command-line tool to manage Frappe Deployments, this tool has various commands, the frappe uses the bench for the command-line tool as well as for the bench directory, so don’t confuse yourself. We will first install the bench package which will be used to set up a frappe environment, create a site, do backup, change the setup, and so on. In short, Bench provides a user-friendly interface to set up and manage multiple frappe-based applications and sites where erpnext is one of the applications.

Now let us install the bench

    sudo -H pip3 install frappe-bench

It will install a bench and will give you a message that the bench is installed successfully, now you can use various bench commands. Starting with the command "bench".
    
    bench --version
    
### Initilize the frappe bench & Install Frappe-Bench Environment using bench CLI 
Let us now create the frappe-bench environment. Here you have to decide the purpose for which you are installing ERPNext, it is just for test or training then you can use the latest version, which will be developing and may not be stable. However you can also use a stable version by choosing a specific version, You can search and learn which is the stable version today.
    
To choose a specific stable version (for Production) you can use the branch version. I will be using branch version 14 in this installation. You can look for the latest stable release of the frappe environment.

    bench init frappe-bench --verbose --frappe-branch version-14

Now frappe bench environment is installed using bench CLI.

Now you can use various bench commands by changing the directory. So you can need to change the directory.
    
    cd frappe-bench/

Once you type "bench" you will see the various commands that bench cli has. Don’t worry, we will not be using all the commands, we just need to install EPRNext but have a quick look at these commands.
    
    bench version
    
### Create a site in frappe bench 
Now you have the application installed in your environment. The next step is to install the application on-site, but before that, we need to create a new site.
    
    bench new-site <sitename> --db-name <dbname>
    eg.
    bench new-site erp.com --db-name erpdb

Now site is deployed, by default frappe application will be installed at site. Don’t open the site yet, because we need to install ERPNext to the site.

### Install ERPNext latest version in bench & site
    
    bench get-app payments
    
    bench get-app erpnext --branch version-14
                        Or
    bench get-app https://github.com/frappe/erpnext --branch version-14

    bench --site erp.com install-app erpnext

    bench start 

Now ERPNext is installed in your server and you are ready to configure it. But beofre configuring there are few more steps in case you want o use this for production.

### Production Deployment
We will use an automatic bench set up for production by using the below command.

    sudo bench setup production erpnext
    bench restart
    
If getting [Error 13] on bench restart
error: <class 'PermissionError'>, [Errno 13] Permission denied: file: /usr/lib/python3/dist-packages/supervisor/xmlrpc.py line: 560
Edit the supervisord configuration file (/etc/supervisor/supervisord.conf)

    sudo nano /etc/supervisor/supervisord.conf

```chmod=0770 ; socket file mode (default 0700)```

```chown=erpnext:erpnext```
    
    sudo service supervisor restart
    
  Open the 0.0.0.0 or server IP in web browser and login to production server
    
### If _js_ and _css_ file is not loading on login window run the following command
    
    sudo chmod o+x /home/erpnext
    
  
### Setup Multitenancy (DNS)
  Assuming that you've already got your first site running and you've performed the production deployment steps, this section explains how to host your   second site (and more). Your first site is automatically set as default site. You can change it with the command,
  
    bench use sitename

To make a new site under DNS based multitenancy, perform the following steps.

    Switch on DNS based multitenancy (once)

      bench config dns_multitenant on

    Create a new site

      bench new-site site2name

    Re generate nginx config

      bench setup nginx

    Reload nginx

      sudo service nginx reload
    
### SSL Installation NginX
