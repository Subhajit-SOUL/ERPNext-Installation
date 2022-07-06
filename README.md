# ERPNext-Installation-Guide
## The complete guide to install ERPNext in your Ubuntu system

### Pre-Requisites 

      Python 3.6+
      Node.js 14+
      Redis 5                                       (caching and real time updates)
      MariaDB 10.3.x / Postgres 9.5.x               (to run database driven apps)
      yarn 1.12+                                    (js dependency manager)
      pip 20+                                       (py dependency manager)
      wkhtmltopdf (version 0.12.5 with patched qt)  (for pdf generation)
      cron                                          (bench's scheduled jobs: automated certificate renewal, scheduled backups)
      NGINX                                         (proxying multitenant sites in production)



### STEP 1 Install git
Git is the most commonly used version control system. Git tracks the changes you make to files, 
so you have a record of what has been done, and you can revert to specific versions should you ever need to. 
Git also makes collaboration easier, allowing changes by multiple people to all be merged into one source.
    
    sudo apt-get install git

### STEP 2 Install python-dev
python-dev is the package that contains the header files for the Python C API, 
which is used by lxml because it includes Python C extensions for high performance.

    sudo apt-get install python3-dev

### STEP 3 Install setuptools and pip (Python's Package Manager).
Setuptools is a collection of enhancements to the Python distutils that allow developers 
to more easily build and distribute Python packages, especially ones that have 
dependencies on other packages. Packages built and distributed using setuptools 
look to the user like ordinary Python packages based on the distutils.

pip is a package manager for Python.  It's a tool that allows you to install and manage 
additional libraries and dependencies that are not distributed as part of the standard library.

    sudo apt-get install python3-setuptools python3-pip

### STEP 4 Install virtualenv
virtualenv is a tool for creating isolated Python environments containing their own copy of
python , pip , and their own place to keep libraries installed from PyPI.
It's designed to allow you to work on multiple projects with different dependencies 
at the same time on the same machine.
    
    sudo apt-get install virtualenv

### STEP 5 Install MariaDB 10.3 stable package
MariaDB is developed as open source software and as a relational database it provides an SQL interface 
for accessing data.

open this link

    https://downloads.mariadb.org/mariadb/repositories/#mirror=piconets
 
For ubuntu 20.04

    sudo apt-get install software-properties-common
    sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
    sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://ftp.icm.edu.pl/pub/unix/database/mariadb/repo/10.3/ubuntu focal main'
    sudo apt update
    sudo apt install mariadb-server
    
 For ubuntu 18.04
 
    sudo apt-get install software-properties-common dirmngr apt-transport-https
    sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
    sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirrors.aliyun.com/mariadb/repo/10.3/ubuntu bionic main'
    sudo apt update
    sudo apt install mariadb-server
     
IMPORTANT :During this installation you'll be prompted to set the MySQL root password.
If you are not prompted for the same You can initialize the MySQL server setup by executing 
the following command
    
    sudo mysql_secure_installation
    
### STEP 6  MySQL database development files

    sudo apt-get install libmysqlclient-dev

### STEP 7 Edit the mariadb configuration ( unicode character encoding )

    sudo nano /etc/mysql/my.cnf

Add this to the my.cnf file

    [mysqld]
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci

    [mysql]
    default-character-set = utf8mb4

Now press (Ctrl-X) to exit

    sudo service mysql restart

### STEP 8 Install Redis
Redis is an open source (BSD licensed), in-memory data structure store, used as a database, 
cache, and message broker.
    
    sudo apt-get install redis-server

### STEP 9 Install Node.js 14.X package
Node.js is an open source, cross-platform runtime environment for developing server-side and 
networking applications. Node.js applications are written in JavaScript, and can be run within the Node.js
runtime on OS X, Microsoft Windows, and Linux.

    sudo apt-get install curl
    curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
    sudo apt-get install -y nodejs

### STEP 10  Install Yarn
Yarn is a JavaScript package manager that aims to be speedy, deterministic, and secure. 
See how easy it is to drop yarn in where you were using npm before, and get faster, more reliable installs.
Yarn is a package manager for JavaScript.
    
    sudo npm install -g yarn

### STEP 11 Install wkhtmltopdf
Wkhtmltopdf is an open source simple and much effective command-line shell utility that enables 
user to convert any given HTML (Web Page) to PDF document or an image (jpg, png, etc)

    sudo apt-get install xvfb libfontconfig wkhtmltopdf
    

### Follow STEP 12 if the root user is the only user in the system continue from STEP 13

### STEP 12 Create a new user

    sudo adduser erpnext
    sudo usermod -aG sudo erpnext
    su - erpnext
    
### STEP 13 Install frappe-bench

    sudo -H pip3 install frappe-bench

IMPORTANT: you may wish to log out and log back into your terminal 
before next step and You must login.
    
    bench --version
    
### STEP 14 Initilize the frappe bench & install frappe latest version 

    bench init frappe-bench --frappe-branch version-13
    
    cd frappe-bench/
    bench start
    
### STEP 15 Create a site in frappe bench 
    
    bench --site <sitename> --db-name <dbname>
    eg.
    bench new-site erp.soulunileaders.com --db-name erpdb

### STEP 16 Install ERPNext latest version in bench & site

    bench get-app erpnext --branch version-13
    ###OR
    bench get-app https://github.com/frappe/erpnext --branch version-13

    bench --site erp.soulunileaders.com install-app erpnext
    
    bench start
#
#
#
#
### Optional step for creation production setup

### Step 17 Setup production
    
    sudo bench setup production erpnext
    bench restart
    
  Open the 0.0.0.0 or server IP in web browser and login to production server
    
    
    
    
  
### Step 18 Setup Multitenancy
  Assuming that you've already got your first site running and you've performed the production deployment steps, this section explains how to host your   second site (and more). Your first site is automatically set as default site. You can change it with the command,
  
      bench use sitename
  
#### Port based multitenancy

    Switch off DNS based multitenancy (once)

      bench config dns_multitenant off

    Create a new site

      bench new-site site2name

    Set port

       bench set-nginx-port site2name 82

    Re generate nginx config

       bench setup nginx

    Reload nginx

      sudo service nginx reload
      
    Reload supervisor
      sudo service supervisor restart
      
#### DNS based multitenancy
You can name your sites as the hostnames that would resolve to it. Thus, all the sites you add to the bench would run on the same port and will be automatically selected based on the hostname.

To make a new site under DNS based multitenancy, perform the following steps.

      Switch on DNS based multitenancy (once)

            bench config dns_multitenant on

      Create a new site

            bench new-site site2name

      Re generate nginx config

            bench setup nginx

      Reload nginx

            sudo service nginx reload
    
### others

      sudo add-apt-repository ppa:certbot/certbot
      sudo apt update
      sudo apt install python-certbot-nginx
      sudo certbot --nginx -d example.com
      
      ./env/bin/python -m pip install -q -U -e /apps/frappe
    
    
