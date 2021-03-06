# Deploying a Rails app to Ubuntu

This post takes you through a proven path to deploy a Rails app to Ubuntu 18.04. The steps will have all of the required commands with descriptions on what you are doing as well as links to various sources I used.

## Outline of the process

Stand up an Ubuntu instance

Log in a root

Create deploy user

Set up ssh keys, fils and folders

Install Postgresql

Create user for postgres database

Create database for your Rails app

Make your deploy user owner of that database

Set your deploy user password in postgres, so you can use it in Rails database.yml





## Some terms used

	<deploy_user>
		The username of the acct you'll use to do most of this
		We will use <deploy_user> in the instructions below

	<app_name>
		The name of your Rails app
  
	<ubuntu_server_ip>
		The ip address of your Ubuntu server

	<server_name>
		DNS name that you point to your <ubuntu_server_ip> address
		This is done by your host provider, or you can do it yourself through GoogleDomains for example.
		I like the flexibility to do it myself, and [GoogleDomains](https://domains.google.com) is really fast at getting your domain working. They say 48hrs, but it usually works within the hour.

## Links
[digital ocean](https://digitalocean.com)

[google domains](https://domains.google.com)


## Setting up your Ubuntu server on Digital Ocean



Digital Ocean
	Create account, project, droplet, get ip_address
	
	Add your personal id_rsa.pub key to your [Digital Ocean account](https://cloud.digitalocean.com/account/security)

	Login as root or user with sudo ability
		$ssh root@ip_address

	Reset root password
		$ passwd
		
	Create deploy_user account with standard password
		$ adduser deploy
	
	Add deploy to sudoers and test sudo ability
		$ usermod -aG sudo deploy
	su - deploy
	sudo ls -la /root


			Add deploy_user to sudoers group
		Login as deployer_user
			Create a .ssh folder for deploy_user
			Create an authorized_keys file for deploy_user
			Add your personal ssh key to the deploy_user authorized_keys file
			Create id_rsa and id_rsa.pub for deploy_user to deploy code from your github account
			Create a known_hosts file
			Set permissions on the folders and files above so you can login and deploy without pain




Install Postgresql
  - installing postgresql
	- using the postgres user account, logged in to postgresql, to create a <postgres_deploy_user> named the same as <ubuntu_deploy_user>
  - creating a postgresql database that matcnhes your <rails_app_name>
	- setting the password for your 


Add deploy user with standard password, add deploy to sudoers and test sudo ability
adduser deploy
usermod -aG sudo deploy
su - deploy
sudo ls -la /root

## Logged in as <ubuntu_deploy_user> for the remainder of this

$ ssh <ubuntu_deploy_user>@<ubuntu_server_ip_addr>



Create ssh dir, generate keys without passphrase and save to ~/.ssh dir
mkdir ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t rsa -b 4096 -C "deploy@<server or repo or app name>"

Add your personal id_rsa.pub hash to a file called ~/.ssh/authorized_keys

## Ensure that SSH file permission are correct
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 640 ~/.ssh/authorized_keys

make sure you can ssh in with both accounts, root and deploy

## Install Javascript runtime
sudo apt-get update
sudo apt-get install -y nodejs npm

Install Postgresql 
sudo apt install postgresql postgresql-contrib libpq-dev

To verify installation, connect to the PostgreSQL database server via postgres user, and using the psql cmd to print the server version:
sudo -u postgres psql -c "SELECT version();"

Create a postgresql <deploy_user_name>
$ sudo -u postgres createuser --interactive

Set postgresql user password for <deploy_user_name>
(logged into Ubuntu server as postgres user, in a psql prompt): $ ALTER ROLE <deploy_user_name> PASSWORD  'deploy_user_name_postgresql_password';

Create database named as <app_name>
(logged into Ubuntu server as postgres user, in a psql prompt): $ CREATE DATABASE <app_name>;

Change ownership of database named as <app_name> to <deploy_user_name>
(logged into Ubuntu server as postgres user, in a psql prompt): $  ALTER DATABASE <app_name> OWNER TO <deploy_user_name>);

Verify database exists and owned by <deploy_user_name>
(logged into Ubuntu server as postgres user, in a psql prompt): postgres-# \l

postgres=# \l
                               List of databases
     Name     |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
--------------+----------+----------+---------+---------+-----------------------
 harvard_bwel | deploy   | UTF8     | C.UTF-8 | C.UTF-8 | 
 postgres     | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0    | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
              |          |          |         |         | postgres=CTc/postgres
 template1    | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
              |          |          |         |         | postgres=CTc/postgres
(4 rows)

Detailed walkthru here: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04

NOTE: we will use the database, deploy user and password:
	in our production.rb file, in our Rails app,
	 snippet:
	# server-based syntax
	# ======================
	# Defines a single server with a list of roles and multiple properties.
	# You can define all roles on a single server, or split them:
	# server "example.com", user: "deploy", roles: %w{app db web}, my_property: :my_value
	# server "example.com", user: "deploy", roles: %w{app web}, other_property: :other_value
	# server "db.example.com", user: "deploy", roles: %w{db}
	server "bwel-poc.smithwebtek.com", user: "deploy", roles: %{app db web}
	
	in our database.yml file on the Ubuntu server at /home/<deploy_user_name>/<app_name>/shared/config/database.yml
	snippet:
	
	production:
	  adapter: postgresql
	  encoding: unicode
	  host: localhost
	  pool: 5
	  database: <app_name>
	  user: <deploy_user_name>
	  password: <deploy_user_name_postgresql_password>
	


Install RVM and Rubyhttps://rvm.io/rvm/install

walkthru

$ sudo apt install gnupg2

$ gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

$ cd /tmp

$ curl -sSL https://get.rvm.io -o rvm.sh

$ less /tmp/rvm.sh

curl -sSL https://get.rvm.io -o rvm.sh
cat /tmp/rvm.sh | bash -s stable --rails

cat /tmp/rvm.sh | bash -s stable
source /home/deploy/.rvm/scripts/rvm
rvm install <whichever version(s) you need>
rvm --default use <version>
rvm list

Setup rails and gemset
https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rvm-on-ubuntu-18-04

Install nginx and passenger:
https://www.phusionpassenger.com/library/install/nginx/install/oss/bionic/

Installing Passenger + Nginx
on Ubuntu 18.04 LTS (with APT)
This page describes the installation of Passenger through the following operating system or installation method: Ubuntu 18.04 LTS (with APT). Not the configuration you are looking for? Go back to the operating system / installation method selection menu.
Table of contents
	• Step 1: install Passenger packages
	• Step 2: enable the Passenger Nginx module and restart Nginx
	• Step 3: check installation
	• Step 4: update regularly
Step 1: install Passenger packages
These commands will install Passenger + Nginx module through Phusion's APT repository. At this point we assume that you already have Nginx installed from your system repository. If not, you should do that first before continuing.

Copy# Install our PGP key and add HTTPS support for APT
sudo apt-get install -y dirmngr gnupg
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo apt-get install -y apt-transport-https ca-certificates
# Add our APT repository
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bionic main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
# Install Passenger + Nginx module
sudo apt-get install -y libnginx-mod-http-passenger
Step 2: enable the Passenger Nginx module and restart Nginx
Ensure the config files are in-place:

Copy$ if [ ! -f /etc/nginx/modules-enabled/50-mod-http-passenger.conf ]; then sudo ln -s /usr/share/nginx/modules-available/mod-http-passenger.load /etc/nginx/modules-enabled/50-mod-http-passenger.conf ; fi
$ sudo ls /etc/nginx/conf.d/mod-http-passenger.conf
If you don't see a file at /etc/nginx/conf.d/mod-http-passenger.conf; then you need to create it yourself and set the passenger_ruby and passenger_root config options. For example:

Copypassenger_root /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini;
passenger_ruby /usr/bin/passenger_free_ruby;
When you are finished with this step, restart Nginx:

Copy$ sudo service nginx restart

Check nginx install
$ sudo systemctl status nginx.service
$ sudo nginx -t

If you need to install nginx because of errors:
$ sudo apt install nginx


Step 3: check installation
After installation, please validate the install by running sudo /usr/bin/passenger-config validate-install. For example:

Copy$ sudo /usr/bin/passenger-config validate-install
 * Checking whether this Phusion Passenger install is in PATH... ✓
 * Checking whether there are no other Phusion Passenger installations... ✓
All checks should pass. If any of the checks do not pass, please follow the suggestions on screen.
Finally, check whether Nginx has started the Passenger core processes. Run sudo /usr/sbin/passenger-memory-stats. You should see Nginx processes as well as Passenger processes. For example:

Copy$ sudo /usr/sbin/passenger-memory-stats
Version: 5.0.8
Date   : 2015-05-28 08:46:20 +0200
...
---------- Nginx processes ----------
PID    PPID   VMSize   Private  Name
-------------------------------------
12443  4814   60.8 MB  0.2 MB   nginx: master process /usr/sbin/nginx
12538  12443  64.9 MB  5.0 MB   nginx: worker process
### Processes: 3
### Total private dirty RSS: 5.56 MB
----- Passenger processes ------
PID    VMSize    Private   Name
--------------------------------
12517  83.2 MB   0.6 MB    PassengerAgent watchdog
12520  266.0 MB  3.4 MB    PassengerAgent server
12531  149.5 MB  1.4 MB    PassengerAgent logger
...
If you do not see any Nginx processes or Passenger processes, then you probably have some kind of installation problem or configuration problem. Please refer to the troubleshooting guide.
Step 4: update regularly
Nginx updates, Passenger updates and system updates are delivered through the APT package manager regularly. You should run the following command regularly to keep them up to date:

Copy$ sudo apt-get update
$ sudo apt-get upgrade
You do not need to restart Nginx or Passenger after an update, and you also do not need to modify any configuration files after an update. That is all taken care of automatically for you by APT.




Configure SSL if applicable
CertBot method:
https://certbot.eff.org/lets-encrypt/ubuntubionic-apache.html

1. SSH into the server
SSH into the server running your HTTP website as a user with sudo privileges.
2. Add Certbot PPA
You'll need to add the Certbot PPA to your list of repositories. To do so, run the following commands on the command line on the machine:
3. sudo apt-get update
4. sudo apt-get install software-properties-common
5. sudo add-apt-repository universe
6. sudo add-apt-repository ppa:certbot/certbot
7. sudo apt-get update
8. Install Certbot
Run this command on the command line on the machine to install Certbot.
sudo apt-get install certbot python-certbot-nginx
9. Choose how you'd like to run Certbot
10. Either get and install your certificates...
Run this command to get a certificate and have Certbot edit your Nginx configuration automatically to serve it, turning on HTTPS access in a single step.
sudo certbot --nginx
11. Or, just get a certificate
If you're feeling more conservative and would like to make the changes to your Nginx configuration by hand, run this command.
sudo certbot certonly --nginx
12. Test automatic renewal
The Certbot packages on your system come with a cron job or systemd timer that will renew your certificates automatically before they expire. You will not need to run Certbot again, unless you change your configuration. You can test automatic renewal for your certificates by running this command:
sudo certbot renew --dry-run
The command to renew certbot is installed in one of the following locations:
13. /etc/crontab/
14. /etc/cron.*/*
15. systemctl list-timers
16. Confirm that Certbot worked
To confirm that your site is set up properly, visit https://yourwebsite.com/ in your browser and look for the lock icon in the URL bar. If you want to check that you have the top-of-the-line installation, you can head to https://www.ssllabs.com/ssltest/.
check your site's 
17.  https:// at SSL Labs.


Now we need to create a custom Nginx configuration file for our app:
> sudo nano /etc/nginx/sites-available/do-testing
To this file, we need to add a server block that:
• enables listening on port 80
• sets the domain name (in this case the IP address of our droplet)
• enables Passenger
• sets the root to the directory of our app.

server {
  listen 80 default_server;
  server_name <IP ADDRESS of your droplet>;
  passenger_enabled on;
  passenger_app_env production;
  root /home/rails/do-testing/public;
}
Save and exit with CTRL + X, Y, ENTER.
Create a symlink for it:
> sudo ln -s /etc/nginx/sites-available/do-testing /etc/nginx/sites-enabled/do-testing


add ssh key for deploy user to github to allow remote deployment

$ sudo cat ~/.ssh/id_rsa.pub

copy the resulting key to clipboard, example: 
deploy@bwel-poc1:~$ sudo cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDadLVA+F1b41GR+IVvTMJx2UUx5bZXY0hpx1TzAGEiwz0eXEBBEMyh4wW7kBRG44zeVh2OwcHv9U36coPSkqkpIqBcNX9DjqzkZBmzZMh3vA8P5aU8CcVkoZFFhX0nGshfTEKq7A68nxipggSVYz6d6UbMmxMlkgx+LgRVXnfXPd7xpMeAfK+t2OEzA273Znw8sZTkgYxeNLY3gK7Q6yZDU3qcPnaL86XqOAJSukqlCoGdWVNw7l75Juoni2F+WzvR3oFGlDiNc2rdo/JHeB+zhBCMCQ/wlqeu1do8EHASNn/7kzWW3ltjiQrOnGm1SJAYxq8Wyi5Uku7hx0vob9fyX1vkEZuIR6hd/+8IqdibBU4ixgRm+pB1YYq7hvrnU5bpb/R211rQ0MzQx8f49wSk/yDKg8VdsW1GSZStLyRyVQq2kGV55noqxtauSRdygEmmZZXdl2QYnvmsFWQ1wK3F+RCvpicoSrqmHIe/ZECzu5sM2MDmn73EgJuPIRk7O4qFJNn7BguJmewZDwHiOXBsRcqtwsNy2Osa94XXHpZeAdte7oi0p+wtHw2Wf1fSM8PAvfRS4W7B4pbeedEUEF8R0Ga/FnppVZEvq3MMYxlZMRde4H9HeOrF6MQjgTXaTgA8u1baaCRngnfEknUPzk7Vumtu6voS4VMAW3KjvkYmVw== deploy@bwel-poc1.smithwebtek.com

logged in as "deploy" user, create folder for Rails app:
$ mkdir <name_of_app>

configure Rails app for Capistrano 

run cap production deploy to generate folder structure

on server, logged in as 'deploy' user, create and populate 2 files:

~/<app_name_folder>/shared/config/secrets.yml
copy from example
add secret: 

$ rake secret
d80bf8c53c84145ddc366e43c94431a719cb9dba66615432d13f9075b258f10b502d6dcd4b280a8f9c156a444850ef1393bbf88507ebce277d361bc3f673e27b

~/<app_name_folder>/shared/config/database.yml

production:
  adapter: postgresql
  encoding: unicode
  pool: 5
  database: harvard_bwel
  user: deploy
  password: pointer
  host: localhost

run cap production deploy again
$ cap production deploy

cap aborted!
SSHKit::Runner::ExecuteError: Exception while executing as deploy@bwel-poc1.smithwebtek.com: cd /home/deploy/harvard_bwel/releases/20200413161559 && npm install exit status: 127

solution:  sudo apt install npm on server


Install Monit
https://cloudwafer.com/blog/installing-monit-on-ubuntu/
You can read more on the Monit Documentation hereYou can also check out the guide on Installing Monit on CentOS here

sudo apt-get update && sudo apt-get upgrade

sudo apt install monit

sudo monit

deploy@bwel-poc1:~$ sudo monit
Monit daemon with PID 20789 awakened


sudo systemctl status monit
deploy@bwel-poc1:~$ sudo systemctl status monit
● monit.service - LSB: service and resource monitoring daemon
   Loaded: loaded (/etc/init.d/monit; generated)
   Active: active (running) since Thu 2020-04-16 14:22:25 UTC; 38s ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 1 (limit: 1152)
   CGroup: /system.slice/monit.service
           └─20789 /usr/bin/monit -c /etc/monit/monitrc

Apr 16 14:22:25 bwel-poc1 systemd[1]: Starting LSB: service and resource monitoring daemon...
Apr 16 14:22:25 bwel-poc1 monit[20775]:  * Starting daemon monitor monit
Apr 16 14:22:25 bwel-poc1 monit[20775]:    ...done.
Apr 16 14:22:25 bwel-poc1 systemd[1]: Started LSB: service and resource monitoring daemon.


deploy@bwel-poc1:~$ sudo monit -t
Control file syntax OK


deploy@bwel-poc1:~$ sudo monit status
Monit: the monit HTTP interface is not enabled, please add the 'set httpd' statement and use the 'allow' option to allow monit to connect



deploy@bwel-poc1:/etc/monit$ sudo service monit restart
deploy@bwel-poc1:/etc/monit$ sudo monit -t
Control file syntax OK

add monit config files for the services you want to monit
 

websocket_rails
	add a websocket_rails file under /etc/monit/conf.d/websocket_rails

		sudo nano  /etc/monit/conf.d/websocket_rails


check process websocket_rails with pidfile /home/deploy/harvard_bwel/current/tmp/pids/websocket_rails.pid
  start program = "/bin/bash -lc 'cd /home/deploy/harvard_bwel/current && bundle exec rake websocket_rails:start_server RAILS_ENV=production'"
  stop program = "/bin/bash -lc 'cd /home/deploy/harvard_bwel/current && bundle exec rake websocket_rails:stop_server RAILS_ENV=production'"

for RVM, you need to point to this path for start / stop commands to run:
check process websocket_rails with pidfile /home/deploy/harvard_bwel/current/tmp/pids/websocket_rails.pid
  start program = "/home/deploy/.rvm/bin/rvm-shell -c 'cd /home/deploy/harvard_bwel/current && bundle exec rake websocket_rails:start_server RAILS_ENV=production'" as uid deploy
  stop program = "/home/deploy/.rvm/bin/rvm-shell -c 'cd /home/deploy/harvard_bwel/current && bundle exec rake websocket_rails:stop_server RAILS_ENV=production'" as uid deploy
  restart program = "/bin/bash -lc 'cd /home/deploy/harvard_bwel/current && RAILS_ENV=production bundle exec rake websocket_rails:stop_server && RAILS_ENV=production bundle exec websocket_rails:start_server '" as uid deploy




deploy@bwel-poc1:~$ sudo systemctl enable monit
monit.service is not a native service, redirecting to systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable monit

run bundle install at project root, to make sure websocket_rails gem has run


Install Redis
https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04

sudo apt update 
sudo apt install redis-server
sudo nano /etc/redis/redis.conf
/etc/redis/redis.conf
. . .
# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised systemd
. . .

sudo systemctl restart redis.service

configure websocket_rails

configure nginx

configure memcache
https://linuxize.com/post/how-to-install-memcached-on-ubuntu-18-04/

https://github.com/memcached/memcached/wiki







