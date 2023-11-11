# How To Install ERPNext 15 in Ubuntu 22.04 LTS
A complete Guide on How to Install Frappe/ERPNext version 15 in Ubuntu 22.04 LTS

### Steps
    Install ERPNext
    Setup Production
    Setup Multitenancy
    Add a Domain
    Instal SSL Certificate
    
### Pre-requisites
    Ubuntu 22.04 LTS
    Python 3.11+
    Node.js 20+
    A user with sudo privileges
    MariaDB 10.3.x
      
### STEP 1 ~ Update and Upgrade Packages
    sudo apt-get update -y
    sudo apt-get upgrade -y
    
### STEP 2 ~ Create a New User
    sudo adduser [frappe-user]
    usermod -aG sudo [frappe-user]
    su [frappe-user]
    cd /home/[frappe-user]

Ensure you have replaced [frappe-user] with your username. eg. sudo adduser frappe

## Install Required Packages
For setting up ERPNext 15, we need to install several software packages first.

Install Python
ERPNext version 15 requires Python version 3.11+.

### STEP 3 ~ Adding PPA repository and Install Python 3.11

    sudo add-apt-repository ppa:deadsnakes/ppa

    sudo apt-get install python3-dev python3.11-dev python3-setuptools python3-pip python3-distutils

### STEP 4 ~ Install Python Virtual Environment

    sudo apt install python3.10-venv python3.11-venv 

### STEP 5 ~ Set Default Python Version to 3.11

    sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.11 1

### STEP 6 ~ Install Redis Server

    sudo apt-get install redis-server

### STEP 7 ~ Install and Setup MariaDB

    sudo apt-get install software-properties-common
    sudo apt install mariadb-server mariadb-client
    sudo mysql_secure_installation

Upon running the last command, you’ll encounter a series of prompts on the server. Make sure to follow the subsequent steps carefully to ensure the setup is configured properly.

Enter current password for root: (Enter your SSH root user password)

Switch to unix_socket authentication [Y/n]: Y

Change the root password? [Y/n]: Y

Remove anonymous users? [Y/n] Y

Disallow root login remotely? [Y/n] Y

Remove test database and access to it? [Y/n] Y

Reload privilege tables now? [Y/n] Y


### STEP 8 ~ Create my.cnf MYSQL config file
    sudo nano /etc/mysql/mariadb.conf.d/my.cnf
##### Add these lines
    [mysqld]
    innodb-file-format=barracuda
    innodb-file-per-table=1
    innodb-large-prefix=1
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci
    
    [mysql]
    default-character-set = utf8mb4

###  STEP 9 ~ Install other packages
    sudo apt-get install xvfb libfontconfig wkhtmltopdf
    
    sudo apt-get install libmysqlclient-dev
    
###  STEP 10 ~ Install CURL, Node.js, NPM, and Yarn
#### CURL
    sudo apt install curl
##### Node.js
    sudo apt install curl 
    curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
    source ~/.profile
    nvm install 20
##### npm
    sudo apt-get install npm
##### Yarn
    sudo npm install -g yarn
    
### STEP 10 ~ Install Frappe Bench
    sudo pip3 install frappe-bench

### STEP 11 ~ Initialize Frappe Bench
    bench init frappe-folder --frappe-branch version-15
    cd frappe-folder/
    
##### Add the node-sass package
    yarn add node-sass
    
#### Change user directory permissions
    chmod -R o+rx /home/[frappe-user]
    
### STEP 12 ~ Create a New Site
    bench new-site [site-name]
### STEP 13 ~ Install ERPNext and other Apps
    bench get-app erpnext --branch version-15
    
    bench get-app payments --branch version-15
    
    bench --site [site-name] install-app erpnext

#### Lastly
    bench use [site-name]
    
    bench start

# Setting ERPNext for Production
### STEP 1 ~ Enable Scheduler
    bench --site [site-name] enable-scheduler
### STEP 2 ~ Disable maintenance mode
    bench --site [site-name] set-maintenance-mode off
### STEP 3 ~ Setup production config
    sudo bench setup production [frappe-user]
### STEP 4 ~ Setup NGINX to apply the changes
    bench setup nginx
### STEP 5 ~ Restart Supervisor and Launch Production Mode
    sudo supervisorctl restart all
    sudo bench setup production [frappe-user]

# Setup Multitenancy
#### DNS based multitenancy 
You can name your sites as the hostnames that would resolve to it. Thus, all the sites you add to the bench would run on the same port and will be automatically selected based on the hostname.

To make a new site under DNS based multitenancy, perform the following steps.

### STEP 1 ~ Switch on DNS based multitenancy (once)
    bench config dns_multitenant on
    
### STEP 2 ~ Create a new site
    bench new-site site2name
    
### STEP 2 ~ Re generate nginx config
    bench setup nginx

### STEP 3 ~ Reload nginx

    sudo service nginx reload
    
# Adding a Domain with SSL to your Site
### Add Domain
    bench setup add-domain [desired-domain]
### Install Let's Encrypt Certificate
#### Install snapd on your machine
    sudo apt install snapd
#### Update snapd
    sudo snap install core;
    sudo snap refresh core
#### Remove existing installations of certbot
    sudo apt-get remove certbot
#### Install certbot
    sudo snap install --classic certbot

    sudo ln -s /snap/bin/certbot /usr/bin/certbot
#### Get Certificate
    sudo -H bench setup lets-encrypt [site-name]
You will be faced with several prompts, respond to them accordingly. This command will also add an entry to the crontab of the root user (this requires elevated permissions), that will attempt to renew the certificate every month.
