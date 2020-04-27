Deployment
Skip to end of metadata
Created by Smith, Bradley J., last modified less than a minute ago
Go to start of metadata
Overview

This document describes the deployment process for the BWEL application to Ubuntu 18.04

This version does not use Taperole / ansible solution from the original app.

The stack is: Ubuntu 18.0, Nginx, Passenger, Redis-server, Monit



SSH in as root or user with sudo ability



Add deploy user with standard password, add deploy to sudoers and test sudo ability

$ adduser deploy
$ usermod -aG sudo deploy
$ su - deploy
$ sudo ls -la /root



As deploy user, create ssh dir, generate keys without passphrase and save to ~/.ssh dir

$ mkdir ~/.ssh
$ chmod 700 ~/.ssh
$ ssh-keygen -t rsa -b 4096 -C "deploy@<server or repo or app name>"



Add your personal id_rsa.pub hash to a file called ~/.ssh/authorized_keys


Ensure that SSH file permission are correct

$ chmod 600 ~/.ssh/id_rsa

$ chmod 644 ~/.ssh/id_rsa.pub

$ chmod 640 ~/.ssh/authorized_keys



Install Javascript runtime

$ sudo apt-get update

$ sudo apt-get install -y nodejs npm



Install Postgresql 

Detailed walkthru here: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04

$ sudo apt install postgresql postgresql-contrib libpq-dev


Verify PostgreSQL installation
connect to the PostgreSQL database server via the postgres user, and use the psql cmd to print the server version:
$ sudo -u postgres psql -c "SELECT version();"

example of correct output:

                                                                version                                                                 

----------------------------------------------------------------------------------------------------------------------------------------

 PostgreSQL 10.12 (Ubuntu 10.12-0ubuntu0.18.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0, 64-bit

(1 row)

create a postgresql <deploy_user_name>
$ sudo -u postgres createuser --interactive

set password for postgresql deploy user <deploy_user_name> ( use same username as deploy user on Ubuntu server)
(logged into Ubuntu server as postgres user, in a psql prompt):

postgres=# ALTER ROLE <deploy_user_name> PASSWORD 'a_password';

create database named same as <app_name>
(logged into Ubuntu server as postgres user, in a psql prompt):
postgres=# CREATE DATABASE <app_name>;

change ownership of the <app_name> database to <deploy_user_name>
(logged into Ubuntu server as postgres user, in a psql prompt):
postgres=# ALTER DATABASE <app_name> OWNER TO <deploy_user_name>);

verify database <app_name> exists and is owned by <deploy_user_name>
(logged into Ubuntu server as postgres user, in a psql prompt):

example of correct output:

postgres=# \l
                               List of databases
     Name     |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
--------------+----------+----------+---------+---------+-----------------------
 harvard_bwel | deploy   | UTF8     | C.UTF-8 | C.UTF-8 | 
 postgres     | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0    | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
              |          |          |         |         | postgres=CTc/postgres
 template1    | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
              |          |          |         |         | postgres=CTc/postgres
(4 rows)

NOTE: There are two files to create on the server, which are not deployed to github: 
     database.yml
     secrets.yml

We will use the database name, user and password in configuring our Rails app with Capistrano,
and in our database.yml file on the Ubuntu server as part of our deployment configuration. (see Rails Capistrano setup below)



Install Ruby and a version manager on Ubuntu (two version managers shown here, choose either RVM or RBENV)


Install RBENV and Ruby

https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-18-04

 – -OR --

Install RVM and Ruby
https://rvm.io/rvm/install



Install Nginx and Passenger

https://www.phusionpassenger.com/library/install/nginx/install/oss/bionic/



check nginx install:

$ sudo systemctl status nginx.service
or

$ sudo service nginx status



check configuration file syntax:

$ sudo nginx -t



If you need to install nginx because of errors:

$ sudo apt install nginx


a sample nginx configuration for an app: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-nginx-app_name-conf

named harvard_bwel
using domain name bwel-prod.smithwebtek.com
located at /etc/nginx/sites-enabled/harvard_bwel
sym linked with command: 
$ sudo ln -s /etc/nginx/sites-enabled/harvard_bwel /etc/nginx/sites-available/harvard_bwel
with a location configured for a websocket path
with a location configured for /assets
with ssl set up using cert bot ( see below )
where root is pointing to the /public folder that results from using Capistrano to deploy Rails app


Configure SSL if applicable

   CertBot method: https://certbot.eff.org/lets-encrypt/ubuntubionic-apache.html

   NOTES:

choose NGINX: Follow the steps at that link, be sure to choose NGINX in drop down.
domain name: If you do not have a domain name setup yet, you can still use your server IP address in the cert bot diaglogue
allow cert bot to edit your config file: After installing cert bot, you'll have a dialogue at the command line to allow nginx to write in redirects

Create a deployment folder for your Rails app

Log in as the deploy user
$ mkdir <app_name>


Rails Capistrano setup

This link provides several related processes, focus on the Capistrano section (Step 6 — Adding Deployment Configurations in the Rails App):

https://www.digitalocean.com/community/tutorials/deploying-a-rails-app-on-ubuntu-14-04-with-capistrano-nginx-and-puma

Start by adding these lines to the Gemfile in the Rails App:

Gemfile

gem 'capistrano', '~> 3.11'
gem 'capistrano-rails', '~> 1.4'
gem 'capistrano-passenger', '~> 0.2.0'
gem 'capistrano-rbenv', '~> 2.1', '>= 2.1.4' # or capistrano-rvm if preferred
# gem 'capistrano-rvm'
gem 'capistrano-websocket-rails'
gem "ed25519", ">= 1.2"
gem "bcrypt_pbkdf", ">= 1.0"


Run bundler
$ bundle install


Install Capistrano
After bundling, run the following command to configure Capistrano:
$ cap install

This creates 4 files.
Configure Capistrano by editing these files:

  Capfile

    example: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-capfile-rb


  config/deploy/production.rb

    example: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-production-rb


  config/deploy/staging.rb

    example (see production.rb for example)


  config/deploy.rb

    example: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-deploy-rb



Run cap production deploy

The deploy will fail, but it will create the folder structure you need on Ubuntu server, where you can create database.yml and secrets.yml


Create the files needed on Unbuntu server, which Rails app requires, and which you need present for successful deployment:

    /home/deploy/<app_name>/shared/config/database.yml

        example: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-database-yml

   
    /home/deploy/<app_name>/shared/config/secrets.yml

        NOTE: secrets.yml will need a 'SECRET_KEY_BASE' value, if it is not present in ENV

        On local machine, at the root of Rails app, run this command, to get the long string that follows:

       $ rake secret

      d80bf8c53c84145ddc366e43c94431a719cb9dba66615432d13f9075b258f10b502d6dcd4b280a8f9c156a444850ef1393bbf88507ebce277d361bc3f673e27b

      Add the outputted long string to secrets.yml on Ubuntu server at:
          /home/<deploy_user>/shared/config/secrets.yml as seen in this example:

           https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-secrets-yml



Run cap production deploy again

    deal with errors such as:

    precompile assets fails





assets not appearing in browser	$ cd /home/<deploy_user>/current
$ RAILS_ENV=production bundle exec rake assets:precompile
$ npm install
repo not found	check config/deploy.rb for correct :repo_url
adapter not specified

rake aborted!

ActiveRecord::AdapterNotSpecified: 'development' database is not configured. Available: ["production"]

check that database.yml is formatted correctly, spaces and indentation matter
it is also possible to get errors from pasting in text to a yml file, type it if you have doubts
what is being monitored?

$ sudo monit status
what ports are in use?

sudo lsof -i -P -n | grep LISTEN

start websocket_rails

$ cd /home/deploy/harvard_bwel/current
$ bundle exec rake websocket_rails:start_server RAILS_ENV=production

should see:

Websocket Rails Standalone Server listening on port 3001

twilio calls	
There are 4 things that must be correct for Twilio calls to work


websocket_rails must be running (see above)
TwiML app request url should point to the correct route in BWEL app




3. Twilio phone number should point to the correct route in BWEL app





4.  /home/<deploy_user>/<app_name>/shared/config/secrets.yml needs Twilio API credentials:
 for example:


defaults: &defaults
  domain: "https://bwel-poc1.smithwebtek.com"
  twilio:
    account_sid: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
    auth_token: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
    sid: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
    secret: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
    phone_number: '+16173973501'
    phone_number_sid: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
    app_sid: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'








Install Redis

guide: https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04





prep for redis install	$ sudo apt update
install redis	$ sudo apt install redis-server
edit redis.conf	$ sudo nano /etc/redis/redis.conf
find this line: 

   supervised no

change to:

       supervised systemd

stop redis server	$ sudo systemctl stop redis
start redis server	$ sudo systemctl start redis
configure for monit (see monit section)	


Install Memcached

installation guides: 

https://linuxize.com/post/how-to-install-memcached-on-ubuntu-18-04/

https://github.com/memcached/memcached/wiki





prep for memcached install	$ sudo apt update
install memcached	$ sudo apt install memcached










Install Monit

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-monit

https://cloudwafer.com/blog/installing-monit-on-ubuntu/

You can read more on the Monit Documentation here
You can also check out the guide on Installing Monit on CentOS here
 





prepare to install monit	$ sudo apt-get update && sudo apt-get upgrade
install monit	
$ sudo apt install monit

edit the monitrc file	$ sudo nano /etc/monit/monitrc
find this line: 
   supervised no

change to:
   supervised systemd
set http web_interface settings in monitrc	
this enables access to monit services via web_interface

$ cd /etc/monit/monitrc

find this code:

#set httpd port 2812 and
#     use address localhost  # only accept connection from localhost
#     allow localhost        # allow localhost to connect to the server and
#     allow admin:monit      # require user 'admin' with password 'monit'

change it to:

set httpd port 2812
    allow 0.0.0.0/0.0.0.0  # allow localhost to connect to the server and
    allow admin:monit      # require user 'admin' with password 'monit'

$ sudo service monit restart
navigate to:  http://<server_name>:2812






start monit	$ sudo monit
   Monit daemon with PID 20789 awakened
check monit status	
$ sudo systemctl status monit
-- or --
$ sudo monit status
-- or --
$ sudo service monit status

check syntax of monit control file	
$ sudo monit -t

Control file syntax OK
add process to monitor: websocket_rails	
add a websocket_rails file at: /etc/monit/conf.d/websocket_rails

$ sudo nano  /etc/monit/conf.d/websocket_rails

example file: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-websocket_rails-config-for-monit

you might need to run bundle install at project root if websocket_rails doesn't work/start

$ cd /home/<deploy_user>/<app_name>/current
$ RAILS_ENV=production bundle exec bundle install
$ RAILS_ENV=production bundle exec rake websocket_rails:stop_server
$ RAILS_ENV=production bundle exec rake websocket_rails:start_server

should see:

Websocket Rails Standalone Server listening on port 3001
add process to monitor: nginx	
add an nginx file at: /etc/monit/conf.d/nginx

$ sudo nano  /etc/monit/conf.d/nginx

example: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-nginx-config-for-monit

add process to monitor: redis	
add a redis file at: /etc/monit/conf.d/redis

$ sudo nano  /etc/monit/conf.d/redis

example: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-redis-config-for-monit

add process to monitor: memcached	
add a memcached file at: /etc/monit/conf.d/memcached

$ sudo nano  /etc/monit/conf.d/memcached

example: https://gist.github.com/smithWEBtek/57b85023660aa8e08824c2f137657b4c#file-example-memcached-config-for-monit





