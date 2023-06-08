#        Master Master Postgresql Replication

## Master Server 1 : 192.168.122.5
## Master Server 2 :192.168.122.6

first update and then install some requirments both server (maseter server 1 and 2 )

#### sudo apt install wget gnupg2 lsb-release curl apt-transport-https ca-certificates

Next, enter the 'curl' command below to download the PostgreSQL repository GPG key, convert the .asc file to .gpg via the 'gpg --dearmor' command, then add the PostgreSQL repository.

#### curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | sudo tee /usr/share/keyrings/pgdg.gpg > /dev/null 2>&1

#### sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

update and then install specific version

#### sudo apt install postgresql-15

#### sudo systemctl is-enabled postgresql

#### sudo systemctl status postgresql

you will also need to install the PostgreSQL extension 'plperl', which will be needed by the Bucardo software. Enter the following 'apt install' command to install the 'plperl' extension.

#### sudo apt install postgresql-plperl-15

Now we create bucado user and password and create emty db 

####  cd /var/lib/postgresql

#### sudo -u postgres psql

#####  CREATE USER bucardo WITH SUPERUSER;

##### CREATE DATABASE bucardo OWNER bucardo;List of users on the PostgreSQL server.

List of database on the PostgreSQL server.

#### \l

List of users on the PostgreSQL server.

#### \du

Enter the following query to create a new database 'testdb'. Then, connect to the database 'testdb' via the '\c' query.

#### CREATE DATABASE testdb;

#### \c testdb;

Now enter the following query to create a new table 'users'.

#### CREATE TABLE users (id SERIAL PRIMARY KEY, first_name VARCHAR(255), last_name VARCHAR(255) NOT NULL, city VARCHAR(255) );

Once the table is created, enter the following queries to verify the schema of the table 'users', then verify the list of available data on the table

####  \dt
####  select * from users;

Open the default PostgreSQL configuration '/etc/postgresql/15/main/postgresql.conf' using the following nano editor command.

#### sudo vi /etc/postgresql/15/main/postgresql.conf

Below is the configuration for the master server 1.

######  listen_addresses = 'localhost, 192.168.122.5'
##
Below is the configuration for the master server 2.

######  listen_addresses = 'localhost, 192.168.122.6'

Next, open the default PostgreSQL authentication config file '/etc/postgresql/15/main/pg_hba.conf' using the following nano editor command do this in  master server 1

#### sudo vi /etc/postgresql/15/main/pg_hba.conf
###### #local connection and bucardo user
###### local    all             all                                    trust
###### local    all             bucardo                                trust
###### #Bucardo user remote connections
###### host    all             postgres         192.168.122.6/24       trust
###### host    all             bucardo          192.168.122.6/24       trust

Below is the configuration for the master server 2 . Be sure to change the IP address with the IP address of the  master server 1.

#### sudo vi /etc/postgresql/15/main/pg_hba.conf
###### #local connection and bucardo user
###### local    all             all                                    trust
###### local    all             bucardo                                trust
###### #Bucardo user remote connections
###### host    all             postgres         192.168.122.5/24       trust
###### host    all             bucardo          192.168.122.5/24       trust

Now enter the following systemctl command utility to restart the PostgreSQL service and apply the changes of both server 

#### sudo systemctl restart postgresql

Installing Bucardo in master 1 server 

#### sudo apt install make libdbix-safe-perl libboolean-perl libdbd-mock-perl libdbd-pg-perl libanyevent-dbd-pg-perl libpg-hstore-perl libpgobject-perl libpod-parser-perl libencode-locale-perl

Now download the Bucardo source code via the wget command below.

#### wget -q https://bucardo.org/downloads/Bucardo-5.6.0.tar.gz

Once downloaded, extract the Bucardo source code, the move it to the Bucardo working directory.

#### tar -xvf Bucardo-5.6.0.tar.gz

####  sudo mv Bucardo-5.6.0 /opt/Bucardo

####  cd /opt/Bucardo

Now enter the following command to compile and install Bucardo 

####  perl Makefile.PL

####  sudo make install

For version check 

##### which bucardo

#### bucardo --version

Before you start, enter the following command to create a new data and log directory for Bucardo.

#### sudo mkdir -p /var/run/bucardo /var/log/bucardo
#### touch /var/log/bucardo/log.bucardo

Automatic start when server reboot do this 

#### sudo vi /etc/rc.local

###### #!/bin/sh -e
###### mkdir -p /var/run/bucardo
###### bucardo start
###### exit 0

#### bucardo install

Enter the following command to define the database server and database name that will be replication. That information will be stored as the 'server1' for the PostgreSQL server master server 1 and 'server2' for the master server 2 

#### bucardo add database server1 dbname=testdb host=192.168.122.5
#### bucardo add database server2 dbname=testdb host=192.168.122.6

for single tables add do this 

#### bucardo add table public.users db=server1
#### bucardo add table public.users db=server2

for all tables add to do this in relgroup

#### bucardo add all tables --her=testdbSrv1 db=server1
#### bucardo add all tables --her=testdbSrv2 db=server2

for chcke relgroup
#### bucardo list relgroup
With the relgroup and table added, you will now start the synchronization process for both master servers. Enter the following command to create a new sync 'testdbSrv1' that will sync the source ='server1' and target='server2'. And the sync is called 'testdbSrv2' which will sync between source='server2' and target='server1'.

#### bucardo add sync testdbSrv1 relgroup=testdbSrv1 db=server1:source,server2:target
#### bucardo add sync testdbSrv2 relgroup=testdbSrv2 db=server2:source,server1:target

Now verify the list of sync on Bucardo by entering the following comman

#### bucardo list sync

Next, enter the following command to restart the synchronization process. And you should see an output 'Starting Bucardo' when successful.

#### sudo bucardo restart sync

Lastly, verify the synchronization status using the following 'bucardo' command.

#### bucardo status

Verify Multi-Master Replication Master Server 1

#### sudo -u postgres psql
######  \c testdb
###### INSERT INTO users(id, first_name, last_name, city) VALUES  (1, 'Alice', 'Wonderland', 'Sweden'), (2, 'Bob', 'Rista', 'Romania'), (3, 'John', 'Bonas', 'England');
###### select * from users; 

Verify Multi-Master Replication Master Server 2
#### sudo -u postgres psql
######  \c testdb
###### select * from users; 
###### INSERT INTO users(id, first_name, last_name, city) VALUES  (4, 'Ian', 'Gibson', 'Liverpool'), (5, 'Tom', 'Riddle', 'Paris'), (6, 'Jared', 'Dunn', 'New York');
###### select * from users; 
and again varify Multi-Master Replication Master Server 2
#### sudo -u postgres psql
######  \c testdb
###### select * from users;

#when add new table then do following steps
#### sudo -u postgres psql
######  \c testdb
###### CREATE TABLE meterials ( product_no INTEGER  PRIMARY KEY,  name text, price numeric );
###### \dt
###### \q

Multi-Master Replication Master Server 2
#### sudo -u postgres psql
######  \c testdb
###### CREATE TABLE meterials ( product_no INTEGER  PRIMARY KEY,  name text, price numeric );
###### \dt
###### \q 

Multi-Master Replication Master Server 1 do this src=master server 1 to target= master sarver 2 
#### bucardo list sync 
#### bucardo add all tables --her=testdbSrv1 db=server1
#### bucardo list dbgroup
#### bucardo update sync testdbSrv1  relgroup=testdbSrv1 dbs=testdbSrv1

Multi-Master Replication Master Server 1 do this src=master server 2 to target= master sarver 1 
#### bucardo list sync 
#### bucardo add all tables --her=testdbSrv2 db=server2
#### bucardo list dbgroup
#### bucardo update sync testdbSrv2  relgroup=testdbSrv2 dbs=testdbSrv2
#### sudo bucardo restart sync


Verify Multi-Master Replication Master Server 1

#### sudo -u postgres psql
######  \c testdb
###### INSERT INTO meterials (product_no, name, price) VALUES (1, 'Bread', 50), (2, 'Milk', 95);
###### select * from meterials; 

Verify Multi-Master Replication Master Server 2
#### sudo -u postgres psql
######  \c testdb
###### select * from meterials; 
###### INSERT INTO meterials (product_no, name, price) VALUES (3, 'Water', 35), (4, 'Rice', 80);
###### select * from meterials; 
###### \q

Verify Multi-Master Replication Master Server 1

#### sudo -u postgres psql
######  \c testdb
###### select * from meterials; 
###### \q

#when add new table then do following steps annathor method 

Multi-Master Replication Master Server 1
#### sudo -u postgres psql
######  \c testdb
###### CREATE TABLE products ( product_no INTEGER  PRIMARY KEY,  name text, price numeric );
###### \dt
###### \q


Multi-Master Replication Master Server 2
#### sudo -u postgres psql
######  \c testdb
###### CREATE TABLE products ( product_no INTEGER  PRIMARY KEY,  name text, price numeric );
###### \dt
###### \q

Multi-Master Replication Master Server 1
#### bucardo list sync
remove sync master server 1 to master server 2 (ha proxy to off to master server 1)
#### bucado remove sync testdbSrv1 
#### bucardo add all tables --her=testdbSrv1 db=server1
#### bucardo add sync testdbSrv1 relgroup=testdbSrv1 db=server1:source,server2:target
#### sudo bucardo restart sync

remove sync master server 2 to master server 1 (ha proxy to off to master server 2)
#### bucado remove sync testdbSrv2
#### bucardo add all tables --her=testdbSrv2 db=server2
#### bucardo add sync testdbSrv2 relgroup=testdbSrv2 db=server2:source,server1:target
#### sudo bucardo restart sync

Verify Multi-Master Replication Master Server 1

#### sudo -u postgres psql
######  \c testdb
###### INSERT INTO products (product_no, name, price) VALUES (1, 'Bread', 50), (2, 'Milk', 95);
###### select * from products; 

Verify Multi-Master Replication Master Server 2
#### sudo -u postgres psql
######  \c testdb
###### select * from products; 
###### INSERT INTO products (product_no, name, price) VALUES (3, 'Water', 35), (4, 'Rice', 80);
###### select * from users; 
###### \q

Verify Multi-Master Replication Master Server 1

#### sudo -u postgres psql
######  \c testdb
###### select * from products; 
###### \q

some others command need for future works 
#### bucardo  list sync
#### bucardo  list relgroup
####  bucardo  list dbgroup
####  bucardo list table
#### bucardo  list sequence
#### bucardo remove sync syncname\







































