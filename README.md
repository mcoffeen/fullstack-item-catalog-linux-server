# Udacity Fullstack Nanodegree - Project 5 - Linux Server Configuration

# Server details
IP address: `52.54.9.161.xip.io`
SSH port: `2200`

# Configuring the Server
#### Create `grader` user
`$ sudo useradd -m -s grader`
#### Give `grader` sudo privileges
`$ sudo usermod -aG sudo grader`
#### Generate key pairs for `grader`
Generate the key pairs on the local machine and copy to authorized_keys file on server
#### Log on to server as `grader`
First, cd to the location where the .ssh folder is stored on the local machine.
`$ ssh grader@52.54.9.161 -i ~/.ssh/grader`
#### Disable root login and disable password logins
`$ sudo nano /etc/ssh/sshd_config`
Change line to: "PermitRootLogin no"
Uncomment "PasswordAuthentication no"

Restart the SSH service to allow the changes to take affect.
`$ sudo service ssh restart`
#### Configure the server's local timezone to UTC
`$ sudo timedatectl set-timezone UTC`
#### Change the SSH port from 22 to 2200
`$ sudo nano /etc/ssh/sshd_config`

Now, logins must be done using port 2200 as follows:
`       $ ssh grader@52.54.9.161 -I ~/.ssh/grader -p 2200`
>This will also disable the use of the web server login on the AWS site since it assumes you're using port 22 for SSH.  You'll need to SSH into the server from a shell or other utility.

#### Configure UFW (Uncomplicated Firewall)
Start by blocking all incoming connections:
`$ sudo ufw default deny incoming`
Allow all outgoing connections:
`$ sudo ufw default allow outgoing`
Allow incoming SSH connections on port 2200:
`$ sudo ufw allow 2200/tcp`
>It is also required to allow TCP on port 2200 in the AWS Lightsail network settings.

Allow incoming HTTP connections on port 80 (default port):
`$ sudo ufw allow www`
Allow incoming NTP connections on port 123 (default port):
`$ sudo ufw allow ntp`
Before enabling the firewall, check what rules we created:
`$ sudo ufw show added`
Enable the firewall:
`$ sudo ufw enable`
Verify the firewall is running by checking the status:
`$ sudo ufw status`

#### Install Apache
`$ sudo apt-get install apache2`
#### Install Apache library for WSGI
`$ sudo apt-get install libapache2-mod-wsgi`
#### Install PostgreSQL
`$ sudo apt-get install postgresql postgresql-contrib`

Create a PostgreSQL user `catalog`
`$ sudo -u postgres createuser -P catalog`

Create a database `catalog`
`$ sudo -u postgres createdb -O catalog catalog`
#### Install several packages needed for the web app to function
`$ sudo apt-get install python-psycopg2`
`$ sudo apt-get install python-flask`
`$ sudo apt-get install python-sqlalchemy`
`$ sudo apt-get install python-pip`
`$ sudo pip install oauth2client`
`$ sudo pip install requests`
`$ sudo pip install httplib2`

#### Update all currently installed packages
Update the package indexes
`$ sudo apt-get update`

Upgrade the installed packages
`$ sudo apt-get upgrade`
---
## Modify your app structure to be ready for the deployment
The following documentation was helpful in creating the WSGI application:

See this [Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) for a basic tutorial to get started on deploying a Flask app on Ubuntu including configuring the Apache server.

Follow this page in the Flask documentation describing how to structure the files of the web app:
[Larger Applications](http://flask.pocoo.org/docs/0.12/patterns/packages/)

#### Edit engine in all Python files from SQLite to PostgreSQL
Change engine variable to:
`engine = create_engine('postgresql://catalog:DB-PASSWORD@localhost/catalog')`
Replace DB-PASSWORD with the password created when the DB was created above

#### Google OAuth Updates
In all the Python files, change the path to the client_secrets.json file to make sure it's pointing to the full path name:
`/var/www/catalog/catalog/client_secrets.json`

In the client_secrets.json file, change the javascript_origins and redirect_URIs:
`javascript_origins":["http://52.54.9.161","http://52.54.9.161.xip.io"]`
`redirect_uris":["http://52.54.9.161.xip.io/login, http://52.54.9.161.xip.io/gconnect"]`

Also, ensure the above changes are reflected in the Google Developers Console.

>XIP.IO was used because Google will not accept public IP addresses as a Redirect URI

#### Push the changes to the remote GitHub repo
---
## On the Ubuntu Server:
#### Install git
`$ sudo apt-get install git`

#### Clone the GitHub repo with the web app
`$ cd /var/www`
`$ sudo mkdir catalog`
`$ sudo chown www-data:www-data catalog/`
`$ sudo -u www-data git clone https://github.com/mcoffeen/fullstack-item-catalog-linux-server.git catalog`

#### Make the .git file inaccessible
`$ sudo nano .htaccess`
Add the line "RedirectMatch 404 /\\.git"
See this page on [Hiding Git Repos on Public Sites](https://davidegan.me/hide-git-repos-on-public-sites/) for more details.

#### Initialize and populate the database
Create the database:
`$ python db_setup.py`
Populate the tables with example collections:
`$ python add_collections.py`

#### Enable Apache to Serve the App
After configuring the catalog.conf file as mentioned in the Digital Ocean tutorial linked above:
disable the default host:
`$ sudo a2dissite 000-default.conf`
enable the catalog host:
`$ sudo a2ensite catalog.conf`
restart the service to make the changes take affect
`$ sudo service apache2 restart`

## Resources Used
[Digital Ocean Flask App tutorial](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
[Flask Documentation - Larger Applications](http://flask.pocoo.org/docs/0.12/patterns/packages/)
[Article on XIP.IO for web testing](https://www.wired.com/2012/06/simplify-your-website-testing-with-xip-io/)
Several GitHub repositories of similar projects to discover missed details in documentation.