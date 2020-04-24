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



Check nginx install

$ sudo systemctl status nginx.service
or

$ sudo service nginx status



check configuration file syntax:

$ sudo nginx -t



If you need to install nginx because of errors:

$ sudo apt install nginx


a sample nginx configuration for an app

named harvard_bwel
using domain name bwel-prod.smithwebtek.com
located at /etc/nginx/sites-enabled/harvard_bwel
sym linked with command: 
$ sudo ln -s /etc/nginx/sites-enabled/harvard_bwel /etc/nginx/sites-available/harvard_bwel
with a location configured for a websocket path
with a location configured for /assets
with ssl set up using cert bot ( see below )
where root is pointing to the /public folder that results from using Capistrano to deploy Rails app


server {

  listen 80;

  server_name bwel-prod.smithwebtek.com;

  root /home/deployer/harvard_bwel/current/public;



  listen 443 ssl;

  ssl_certificate /etc/letsencrypt/live/bwel-prod.smithwebtek.com/fullchain.pem; # managed by Certbot

  ssl_certificate_key /etc/letsencrypt/live/bwel-prod.smithwebtek.com/privkey.pem; # managed by Certbot



  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot



  location /websocket {

    proxy_pass http://localhost:3001/websocket;

    proxy_http_version 1.1;

    proxy_set_header Upgrade $http_upgrade;

    proxy_set_header Connection "upgrade";

    proxy_set_header Host $host;

  }



  location /assets {

    root /home/deployer/harvard_bwel/current/public;

    gzip_static on;

    expires max;

    add_header Cache-Control public;

  }



  passenger_enabled on;

  passenger_ruby /home/deployer/.rbenv/shims/ruby;

  passenger_app_env production;



  location /calls/connect {

    passenger_app_group_name harvard_bwel;

    passenger_force_max_concurrent_requests_per_process 0;

  }

}



Configure SSL if applicable

CertBot method: https://certbot.eff.org/lets-encrypt/ubuntubionic-apache.html

NOTES:

choose NGINX: Follow the steps at that link, be sure to choose NGINX in drop down.
domain name: If you do not have a domain name setup yet, you can still use your server IP address in the cert bot diaglogue
allow cert bot to edit your config file: After installing cert bot, you'll have a dialogue at the command line to allow nginx to write in redirects


Set up a folder for your Rails app

Log in as the deploy user

Create the folder

Capify your Rails app

Edit these files

Run cap production deploy

It will fail, but it will create the folder structure needed on Ubuntu server

Create the files needed in /home/deploy/<app_name>/shared/config/

database.yml

secrets.yml



Rails Capistrano setup

NOTE: There are two files to create on the server, which are not deployed to github: 
     database.yml
     secrets.yml

We will use the database name, user and password in configuring our Rails app with Capistrano,
and in our database.yml file on the Ubuntu server as part of our deployment configuration. (see Rails Capistrano setup below)