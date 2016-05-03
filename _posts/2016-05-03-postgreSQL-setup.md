---
layout: post
title:  "PostgreSQL setup"
date:   2016-05-3 07:00:00 +0300
categories: tutorials
---

# Introduction

I found my self straggling with database setup most of the time. 
The reason is that I rarely find my self setting up a database.
And every time i'm piecing up the puzzle from the beginning.
For that reason i will try to document the process from start to finish, with the hope that it will save me time the next time around.
 
# Overview

## The basic steps to set up postgreSQL are

* install the database
* set up the database
* allow the database to be accessed from the rest of the world
  
## The files that we will have to edit are

``` bash
sudo nano /etc/postgresql/9.5/main/postgresql.conf
```
With is the main database configuration where you change the number of connection, memory utilization and various other optimazations.
The second and most relevant configuration file to as is 

``` bash
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```
Which is the configuration file for the access related settings of the database.
Here we will tell the server to allow some host to access some databases using some users.

## Useful commands

``` bash
sudo /etc/init.d/postgresql stop
sudo /etc/init.d/postgresql start 
sudo /etc/init.d/postgresql reload
sudo /etc/init.d/postgresql restart

# connect to database demo with the user michael
psql -d demo -U michael
```

# installation

In order to install postgresql in your machine you can ether; install postgres from the package manager of your destitution or [brew.sh][1] if you are on mac os X; or, use the build provided by postgreSQL; or, build it yourself.
i'm going to install the one that is provided by the package manager of my distribution.
I'm using ubuntu server 16.04 LTS so here we go.

whenever you see apt-get you can substitute it with brew, if not otherwise stated.

``` bash
sudo apt-get update
sudo apt-get install postgresql
# or if you need to choose a version 
sudo apt-get install postgresql-9.5
```
This will install postgreSQL to your system.

# Database initial setup 

Now lets connect to the database so that we can do some initial setup

``` bash
sudo -u postgres psql postgres
``` 
Now we are in the postgres command line shell.
First we need to give the default user a password 

``` sql
\password postgres
```
Enter a new password and then once again.
Now lets create a new database 

``` sql
CREATE DATABASE demo;
```
And then create a new user with password. 

``` sql
CREATE USER michael WITH SUPERUSER PASSWORD 'password';
```
You shouldn't really use "password" as a password.
For more details on how to create a user with the correct access rights please refer to the official [documentation][2].
Finally grand all or some rights to the new user as you wish. 

``` sql 
GRANT ALL PRIVILEGES ON DATABASE demo TO michael;
```
Grand documentation can be found [here][3].
Finally we can leave the postgres shell.
  
``` sql
\q
```

# Setting up the database to accept connections from other places

First and easies configuration file we need to update is   

``` bash
sudo nano /etc/postgresql/9.5/main/postgresql.conf
```
Find the line
``` 
# listen_addresses = 'localhost'
```
and change it to 
``` 
listen_addresses = '*' 
```
take notice of the removed # at the beginning of the line.
The second configuration file we need to update is.

```
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```
add this line at the end of the file 

```
host demo michael 0.0.0.0/0 md5
```
What this line says is that some other computer can asses database demo using the user michael from whatever ip matching 0 bits of the ip 0.0.0.0
Think of the 0.0.0.0/0 as * of a regex.
Its better provide a proper ip here for security reasons, but that's not the topic for today.
Hope this helps.

[1]: http://brew.sh
[2]: http://www.postgresql.org/docs/9.5/static/sql-createuser.html
[3]: http://www.postgresql.org/docs/current/static/sql-grant.html
