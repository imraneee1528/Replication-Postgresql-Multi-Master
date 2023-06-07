#        Master Master Postgresql Replication


first update and then install some requirments 

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

#### CREATE USER bucardo WITH SUPERUSER;

#### CREATE DATABASE bucardo OWNER bucardo;
 
