# Project Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
   - [Objective](#objective)
   - [Requirements](#requirements)
3. [Deployment Instructions](#deployment-instructions)
   - [Provisioning Servers with Vagrant](#provisioning-servers-with-vagrant)
   - [Automating Deployment with Bash Script](#automating-deployment-with-bash-script)
   - [Executing the Ansible Playbook](#executing-the-ansible-playbook)
4. [Code Files](#code-files)
   - [Vagrant Configuration File - Vagrantfile](#vagrant-configuration-file---vagrantfile)
   - [Bash Script - deploy.sh](#bash-script---deploysh)
   - [Ansible Playbook - playbook.yml](#ansible-playbook---playbookyml)
   - [Ansible Configuration - ansible.cfg](#ansible-configuration---ansiblecfg)
   - [Ansible Inventory - inventory.ini](#ansible-inventory---inventoryini)
5. [Log Files](#log-files)
   - [Bash Script Log - deploy.log](#bash-script-log---deploylog)
   - [Ansible Playbook Log - ansible.log](#ansible-playbook-log---ansiblelog)
   - [Cron Job Log - uptime.log](#cron-job-log---uptimelog)
6. [Screenshots](#screenshots)
   - [Screenshot of the Master node on virtualbox](#screenshot-of-the-master-node-on-virtualbox)
   - [Screenshots of the laravel application deployed with Bash script on the Master](#screenshots-of-the-laravel-application-deployed-with-bash-script-on-the-master)
   - [Screenshot of the Slave node on virtualbox](#screenshot-of-the-slave-node-on-virtualbox)
 - [Screenshot of Playbook execution](#screenshot-of-playbook-execution)
   - [Screenshots of the laravel application deployed with Ansible on the Slave](#screenshots-of-the-laravel-application-deployed-with-ansible-on-the-slave)
   - [Screenshot of the cronjob](#screenshot-of-the-cronjob)
7. [Usage](#usage)
8. [Important Notes](#important-notes)
9. [Contributing](#contributing)
10. [References](#references)

## [Introduction](introduction)

Welcome to the documentation for the Star Fish project (Deploy LAMP Stack). This documentation provides a comprehensive guide to automating the provisioning and deployment of a LAMP (Linux, Apache, MySQL, PHP) stack using Vagrant, a bash script, Ansible, and a PHP application (Laravel). The project aims to streamline the process of setting up a web server environment for hosting PHP applications in this case a Laravel application, which will be cloned from the official Laravel repository [GitHub Repository for Laravel](https://github.com/laravel/laravel).

## [Project Overview](project-overview)

### [Objective](objective)

The primary objective of this project is to automate the provisioning of two Ubuntu-based servers, referred to as "Master" and "Slave," using Vagrant. The automation involves the following steps:

1. Create a bash script to automate the deployment of a LAMP stack on the "Master" node.
2. Clone a PHP application from GitHub.
3. Install all necessary packages.
4. Configure the Apache web server and MySQL.
5. Ensure the bash script is reusable and readable.

In addition to the bash script, an Ansible playbook is used to:

1. Execute the bash script on the "Slave" node.
2. Create a cron job to check the server's uptime every day at midnight.

It is also important to verify that the PHP application is accessible through the VM's IP address and take a screenshot as evidence.

### [Requirements](requirements)

To complete the project, the following requirements must be met:

1. Submit the bash script and Ansible playbook to a publicly accessible GitHub repository.
2. Document the steps in Markdown files, including screenshots where necessary.
3. Use either the VM's IP address or a domain name as the URL.

## [Deployment Instructions](deployment-instructions)

In this section, we'll cover the steps to deploy the LAMP stack using Vagrant, the bash script, and the Ansible playbook.

### [Provisioning Servers with Vagrant](provisioning-servers-with-vagrant)

To provision servers using Vagrant, we use the `Vagrantfile` provided. This file defines the configuration for both the "Master" and "Slave" servers. It specifies the base box, networking settings, and provisioning steps.

1. **Vagrant Configuration File**: The `Vagrantfile` specifies the configuration for the "Master" and "Slave" servers.

```ruby
# Deployment of two Ubuntu-based servers, named Master and Slave using Vagrant.

Vagrant.configure("2") do |config|
  # Configuration for Master server
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.define "Master" do |master|
    # Master server settings
    # ...
  end

  # Configuration for Slave server
  config.vm.define "Slave" do |slave|
    # Slave server settings
    # ...
  end
end
```

2. **Server Provisioning**: The Vagrant configuration provisions the servers and ensures that SSH is properly configured. It also executes the `deploy.sh` script on the "Master" server.

3. **Network Configuration**: The servers are connected to a private network to allow communication between them.

Detailed instructions for provisioning servers using Vagrant can be found in the provided [Vagrant Configuration File - Vagrantfile](#vagrant-configuration-file---vagrantfile).

### [Automating Deployment with Bash Script](automating-deployment-with-bash-script)

The `deploy.sh` script automates the deployment of the LAMP stack. It performs the following tasks:

1. Add important repositories to the APT package manager.
2. Updates and upgrades installed packages.
3. Installs Apache, MySQL, PHP, and other necessary packages.
4. Configures Apache for a Laravel application.
5. Installs Composer and sets permissions.
6. Clones the Laravel application from GitHub and configures it.
7. Set up the MySQL database and update the `.env` file.
8. Caches configuration values and runs database migrations.

The script also adds firewall rules to allow necessary ports.

The `deploy.sh` script is thoroughly documented in the
- [deploy.sh](#bash-script---deploysh)

### [Executing the Ansible Playbook](executing-the-ansible-playbook)

The Ansible playbook, `playbook.yml`, automates the execution of the `deploy.sh` script on the "Slave" server and sets up a cron job to check server uptime.

1. **Copying Deployment Script**: The playbook copies the `deploy.sh` script to the "Slave" server.

2. **Executing Deployment Script**: It then executes the script and logs the output.

3. **Permissions and Cron Job**: The playbook ensures correct permissions for directories and creates a cron job to check the server's uptime.

The Ansible playbook and its configurations are documented in the

- [Ansible Playbook - playbook.yml](#ansible-playbook---playbookyml)

- [Ansible Configuration - ansible.cfg](#ansible-configuration---ansiblecfg)

- [Ansible Inventory - inventory.ini](#ansible-inventory---inventoryini)

## [Code Files](code-files)

In this section, we provide the content of the code files used in the project.
### [Vagrant Configuration File - Vagrantfile](vagrant-configuration-file---vagrantfile)

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Deployment of two Ubuntu-based servers, named Master and Slave using Vagrant.

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.define "Master" do |master|
    master.vm.hostname = "Master"
    master.vm.network "private_network", ip: "192.168.56.20"
    master.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "1"
    end
    master.vm.provision "shell", inline: <<-SHELL
    ssh_config_file="/etc/ssh/sshd_config"
    sudo sed -i 's/^PasswordAuthentication yes/PasswordAuthentication no/' "$ssh_config_file"
    sudo sed -i 's/^#PubkeyAuthentication yes/PubkeyAuthentication yes/' "$ssh_config_file"
    sudo systemctl restart ssh || sudo service ssh restart
    sudo apt-get install -y avahi-daemon
    SHELL
    master.vm.provision "shell", path: "./ansible-playbook/deploy.sh" end
  config.vm.define "Slave" do |slave|
    slave.vm.hostname = "Slave"
    slave.vm.network "private_network", ip: "192.168.56.21"
    slave.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "2"
    end
    slave.vm.provision "shell", inline: <<-SHELL
      ssh_config_file="/etc/ssh/sshd_config"
      sudo sed -i 's/^PasswordAuthentication yes/PasswordAuthentication no/' "$ssh_config_file"
      sudo sed -i 's/^#PubkeyAuthentication yes/PubkeyAuthentication yes/' "$ssh_config_file"
      sudo systemctl restart ssh || sudo service ssh restart
      sudo apt-get install -y avahi-daemon
    SHELL
  end
end
```

### [Bash Script - deploy.sh](bash-script---deploysh)

```bash
#!/bin/bash

# This script will automatically deploy a LAMP stack and clone a PHP application from a GitHub repository (https://github.com/laravel/laravel.git) and configure the Apache web server and MySQL database.

# Check if the script is being run as root, if not run as root
if [[ "$(id -u)" -ne 0 ]]; then
    sudo -E "$0" "$@"
    exit
fi

# Log all the commands and the output to a file called deploy.log in the shared directory
shared_dir="/vagrant"
log_file="$shared_dir/deploy.log"
exec > >(tee -a "$log_file") 2>&1

# Get server IP address
server_ip=$(ip addr show eth1 | awk '/inet / {print $2}' | cut -d/ -f1)

# Define variables for configurations
server_admin_email="ayodejihamed@gmail.com"
laravel_repo_url="https://github.com/laravel/laravel.git"

# Start logging the script
echo " .......Deployment started at $(date)......."

# Add important repositories to the APT package manager (for Debian/Ubuntu)
echo "...............Adding important repositories to the package manager................."
sudo apt-get install -y software-properties-common
sudo add-apt-repository -y ppa:ondrej/php

# updating the package list to ensure your getting the latest version
echo "..............Updating the package list............."
sudo apt-get update -y

# upgrading the updated packages
echo "............Upgrading the installed packages............."
sudo apt-get upgrade -y


## Install and setup AMP (Apache, MySQL, PHP) and other packages ##

# Install Apache
echo "..................Installing Apache............"
sudo apt install -y apache2

# Start and enable Apache
echo ".............Enabling Apache services..........."
sudo systemctl start apache2
sudo systemctl enable apache2

# Install gnupg to handle GPG keys
sudo apt install -y gnupg

# Generate a random secure password for MySQL root user
echo "............Generating a random secure password for MySQL root user................."
password=$(openssl rand -base64 12)

# Install MySQL Server in a Non-Interactive mode. Default root password will be set to the one you set in the previous step
echo ".........Installing MySQL Server............."
debconf-set-selections <<< "mysql-server mysql-server/root_password password $password"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $password"
apt-get install -y mysql-server

# Display the MySQL root password
echo " MySQL root password: $password "

# Disallow remote root login
echo "...............Disallowing remote root login.........."
sed -i "s/.*bind-address.*/bind-address = 127.0.0.1/" /etc/mysql/mysql.conf.d/mysqld.cnf

# Remove the test database
echo "..........Removing the test database........."
mysql -uroot -p"$password" -e "DROP DATABASE IF EXISTS test;" || true

# Restart MySQL
echo "........... Restarting MySQL..........."
systemctl restart mysql

# Install PHP and some of the most common PHP extensions
echo "...............Installing PHP and some of the most common PHP extensions.............."
apt-get install -y php8.2 libapache2-mod-php8.2 php8.2-common php8.2-mysql php8.2-gmp php8.2-sqlite3 php8.2-curl php8.2-intl php8.2-mb>

# Path to the PHP configuration file
php_ini_file="/etc/php/8.2/apache2/php.ini"

# Configure PHP
echo "............Configuring PHP............"
sudo sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/" "$php_ini_file"

# Uncommenting the sqlite extension
sudo sed -i "s/;extension=sqlite3/extension=sqlite3/" "$php_ini_file"

# Restart Apache web server
echo "..........Restarting Apache web server..........."
sudo systemctl restart apache2

# Install Composer
echo "............ Installing Composer............"
sudo curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer


# Create Apache configuration file for Laravel
echo "............Creating a new Apache configuration file for Laravel............."
sudo tee /etc/apache2/sites-available/laravel.conf > /dev/null <<EOL
<VirtualHost *:80>
    ServerAdmin $server_admin_email
    ServerName $server_ip
    DocumentRoot /var/www/html/laravel/public

    <Directory /var/www/html/laravel>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOL

# Disable the default Apache configuration file
echo "............ Disabling the default Apache configuration file............."
sudo a2dissite 000-default.conf

# Enable the new configuration
sudo a2ensite laravel.conf

# Enable the PHP module in Apache
echo ".............Enabling the PHP module in Apache................"
sudo a2enmod php8.2

# Reload Apache to apply changes
sudo systemctl reload apache2


# Install git if it is not already installed and update it to the latest version
echo "...............Installing git............."
# Check if Git is installed and install/upgrade it
if [ -x "$(command -v git)" ]; then
    echo "Git is already installed. Checking for updates..."
    sudo apt-get update
    sudo apt-get install --only-upgrade git
else
    echo ".......Git is not installed. Installing the latest version..."
    sudo apt-get install -y git
fi

# Navigate to the web root directory
echo "........... Navigating to the web root directory.............."
cd /var/www/html || exit

# Clone the Laravel repository from GitHub
echo "..............Cloning the Laravel repository from GitHub .............."
git clone $laravel_repo_url

# Navigate to the Laravel application directory
echo "..............Navigating to the Laravel application directory..........."
cd laravel || exit

# Install the Laravel application dependencies
echo "................Installing the Laravel application dependencies ............"
composer install --no-interaction --optimize-autoloader --no-dev

# Set Laravel permissions
echo "========== Setting Laravel permissions =========="
sudo chown -R www-data:www-data /var/www/html/laravel
sudo chmod -R 755 /var/www/html/laravel
sudo chmod 775 /var/www/html/laravel/database
sudo chown -R www-data:www-data /var/www/html/laravel/database/database.sqlite
sudo chmod 664 /var/www/html/laravel/database/database.sqlite
sudo chmod -R 755 /var/www/html/laravel/storage
sudo chmod -R 755 /var/www/html/laravel/bootstrap/cache

# reload apache2 to aplly the changes
sudo systemctl reload apache2

# Create a new .env file from the .env.example file
echo "..............Creating a new .env file from the .env.example file..........."
sudo cp .env.example .env

# Generate an application key
echo "...........Generating an application key..........."
php artisan key:generate

# Mysql Database Setup
echo "..........Database Setup........."
db_name="laravel"
mysql_user="root"

mysql -u $mysql_user -p"$password" <<MYSQL_SCRIPT
CREATE DATABASE $db_name;
GRANT ALL PRIVILEGES ON $db_name.* TO '$mysql_user'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
MYSQL_SCRIPT

# Update the .env file with the database connection details
echo "............Updating the .env file with the database connection details........."
sed -i "s/DB_PASSWORD=.*/DB_PASSWORD=\"$password\"/" .env

# Cache the configuration values
echo "...............Caching the configuration values..........."
php artisan config:cache

# Run the database migrations
echo "...........Running the database migrations..........."
php artisan migrate --force

# Restart Apache web server
echo "...........Restarting Apache web server........."
systemctl restart apache2
sudo systemctl reload apache2


# End logging the script
echo "...........Deployment ended at $(date)........."

# settteing database permission
echo ".....stting database permission......."
sudo chown -R www-data:www-data /var/www/html/laravel/database/database.sqlite
sudo chmod 664 /var/www/html/laravel/database/database.sqlite

# reload apache2
sudo systemctl reload apache2


# Add firewall rules

#  Check if ufw is installed and active
if ! dpkg -l | grep -q "ufw"; then
    echo "ufw is not installed. Installing..."
    apt-get install -y ufw
    echo "ufw installed."
fi

# Enable ufw
echo "..........Enabling ufw..........."
ufw --force enable

echo "............. Adding firewall rules........."
ufw allow openssh
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 3306/tcp

# Access the application in a browser using the server IP address
echo " .....Access the application in a browser using the server IP address...."
echo "........Server IP address: $server_ip........"

# End of script

### [Ansible Playbook - site.yml](ansible-playbook---siteyml)
---
- name: Execute Bash Script
  hosts: Slave
  become: yes
  tasks:
    - name: Copy bash script to remote host
      copy:
        src: ./deploy.sh
        dest: /tmp/deploy.sh
        mode: "0755"


    - name: Execute the bash script
      shell: /tmp/deploy.sh

    - name: Ensure correct permissions for /vagrant directory
      file:
        path: /vagrant
        state: directory
        mode: 0755
      when: not ansible_check_mode

    - name: Ensure correct permissions for uptime.log file
      file:
        path: /vagrant/uptime.log
        state: touch
        mode: 0644
      when: not ansible_check_mode

    - name: Create a cron job to check the server's uptime every 12 am
      cron:
        name: "Check server uptime"
        minute: "0"
        hour: "0"
        job: "uptime >> /vagrant/uptime.log"
        user: vagrant

        mode: 0644
      when: not ansible_check_mode

The `site.yml` file automates the deployment and can be found in the provided file.

### [Ansible Configuration - ansible.cfg](ansible-configuration---ansiblecfg)

```ini
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible
```

The `ansible.cfg` file contains the configuration settings for Ansible.
### [Ansible Inventory - inventory.ini](ansible-inventory---inventoryini)

```ini
[master]
Master ansible_host=192.168.8.131 ansible_user=vagrant ansible_connection=ssh
[Slave]
Slave ansible_host=192.168.8.132 ansible_user=vagrant ansible_connection=ssh
```

The `inventory.ini` file defines the servers and their connection details for Ansible.

That concludes the code files used in the project. In the next section, we'll discuss log files and their purposes.

## [Log Files](log-files)

The project generates several log files that provide insights into the deployment and operations:

### [Bash Script Log - deploy.log](bash-script-log---deploylog)

The `deploy.log` file contains detailed logs of the deployment process using the `deploy.sh` script. This log helps in debugging and tracking the execution of the script. The script generates this log to document each step and capture any errors that may occur.

You can find the `deploy.log` file [deploy.log](deploy.log) in the project directory.

### [Ansible Playbook Log - ansible.log](ansible-playbook-log---ansiblelog)

The `ansible.log` file captures the output of the Ansible playbook execution. It logs the tasks performed by the playbook, including copying the `deploy.sh`

 script and executing it on the "Slave" server. This log is essential for troubleshooting and verifying the automation process.

You can find the `ansible.log` file [ansible.log](ansible.log) in the project directory.

### [Cron Job Log - uptime.log](cron-job-log---uptimelog)

The `uptime.log` file records server uptime data. A cron job is scheduled to run daily at 12 am to check and record the server's uptime. The `uptime.log` file accumulates these records over time and serves as evidence of server availability.

You can find the `uptime.log` file [uptime.log](uptime.log) in the project directory.

## [Screenshots](screenshots)

This section contains screenshots of the deployed application and the execution of the Ansible playbook.

### [Screenshot of the Master node on virtualbox](master-node-on-virtualbox)

![Screenshot of the Master node on virtualbox](./screenshots/Master-node-vb.png)

### [Screenshots of the laravel application deployed with Bash script on the Master](screenshots-master-node)
![Screenshots of the laravel application deployed with Bash script on the Master](./screenshots/master-server-laravel.png)

![Screenshots of the laravel application deployed with Bash script on the Master](./screenshots/master-server-laravel2.png)

![Screenshots of the laravel application deployed with Bash script on the Master](./screenshots/master-server-laravel3.png)

![Screenshots of the laravel application deployed with Bash script on the Master](./screenshots/master-server-laravel4.png)

### [Screenshot of the Slave node on virtualbox](slave-node-on-virtualbox)

![Screenshot of the Slave node on virtualbox](./screenshots/Slave-node-vb.png)

### [Screenshot of Playbook execution](playbook-execution-screenshot)

![Screenshot of Playbook execution](./screenshots/Ansible-execution.png)

### [Screenshots of the laravel application deployed with Ansible on the Slave](screenshots-slave-node)

![Screenshots of the laravel application deployed with Ansible on the Slave](./screenshots/slave-server-laravel.png)

![Screenshots of the laravel application deployed with Ansible on the Slave](./screenshots/slave-server-laravel2.png)

![Screenshots of the laravel application deployed with Ansible on the Slave](./screenshots/slave-server-laravel3.png)

![Screenshots of the laravel application deployed with Ansible on the Slave](./screenshots/slave-server-laravel4.png)
### [Screenshot of the cronjob](cronjob-screenshot)

![Screenshot of the cronjob](./screenshots/cronjob.png)

In the next section, we will provide instructions on how to use the project, including the deployment and maintenance of the LAMP stack.

## [Usage](usage)

This section provides instructions on how to use the project to deploy the LAMP stack and maintain the server environment.

To use this project:

1. Clone the project's GitHub repository: [GitHub Repository Link](https://github.com/Hamed-altschool/Cloud-Eng-v2-2nd-Sems-Exam.git).

2. Configure the `Vagrantfile` to set up the desired server configurations and network settings. Customize the parameters as needed for your environment.

3. Provision the servers using Vagrant. Run the following command in the project directory:

```bash
vagrant up
```

4. The provisioning process will set up the "Master" and "Slave" servers with the LAMP stack. Review the logs in the `deploy.log` file for details on the deployment.

5. Access the PHP application by using the VM's IP address or domain name.

6. Use the provided Ansible playbook, `playbook.yml`, to automate the execution of the `deploy.sh` script on the "Slave" server. Run the following command in the project directory:

```bash
ansible-playbook playbook.yml
```

7. The playbook will execute the deployment script and create a cron job to check server uptime.

8. Monitor the logs in the `ansible.log` and `uptime.log` files for the playbook's execution and uptime records.

## [Important Notes](important-notes)

- Regularly check the log files (`deploy.log`, `ansible.log`, and `uptime.log`) for any errors or issues.

- Ensure that you have the necessary credentials and permissions to deploy and configure servers.

- Customization: Adjust the project configurations, such as network settings and server parameters, to suit your specific requirements.

## [Contributing](contributing)
Contributions to this project are highly welcomed. If you would like to contribute, follow these steps:

1. Fork the project's GitHub repository.

2. Create a new branch for your feature or bug fix.

3. Make your changes and commit them to your branch.

4. Submit a pull request to the main repository for review and integration.

## [References](references)

For more information and helpful resources, consider exploring the following references:

- [Vagrant Documentation](https://www.vagrantup.com/docs)
- [Ansible Documentation](https://docs.ansible.com)
- [Laravel PHP Framework](https://laravel.com)
- [GitHub Repository for Laravel](https://github.com/laravel/laravel)
- [GitHub Repository for Vagrant](https://github.com/hashicorp/vagrant)

This concludes the documentation for my Cloud Engineering Second Semester Examination Project. It is my hope that this comprehensive guide assists you in successfully deploying a LAMP stack and managing server automation.

If you have any questions or need further assistance, please don't hesitate to reach out to [me](https://twitter.com/qurtana) or the project contributors.
Thank you for using this project, and best of luck with your cloud engineering endeavors!
