# GUIA INSTALACION POSTGRESQL 10


## 1 UPDATE THE SYSTEM
    sudo apt-get update
    sudo apt-get -y upgrade 

## 2 Import Repository Signing Key
The PostgreSQL team is using GPG to sign downloadable packages from the PostgreSQL website.
We add PostgreSQL public GPG key so apt can verify that packages downloaded from PostgreSQL repository are not tampered or corrupt.

### To Add PostgreSQL GPG public key:
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

## 3 Add PostgreSQL 10 repository
After adding PostgreSQL release keys, we create a new repository configuration for PostgreSQL using the command below.
    echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/postgresql-pgdg.list &gt; /dev/null

Refresh apt metadata using the command below.
    sudo apt-get update 

## 4 Instal- postgresql 10
To install postgresql:
    sudo apt-get install postgresql-10

Now, postgresql is installed.

## Check postgres installation
To verify the installed postgresql packages:
    dpkg -l | grep postgresql

## Check Postgresql listening ports
Postgresql by default listen in port 5432, we can check who is listen in that port:
    sudo netstat -naptu | grep 5432

Also we can check if exists any process listening called postgres in that port 

    sudo netstat -naptu | grep postgres

As alternative we can use ss to check which process is listening in that port:
    sudo ss -atnp | grep 5432
Also we can check if is a deamon called postgres listen in that port:
    sudo ss -atnp | grep postgres

## check postgreql process 
We can use ps command to check which deamon is running with the name postgres
    ps -aux | grep postgres

Check Postgresql 10 resource usage like cpu and memory:
    top -u postgres

## Managing postgresql service

 To check the status of the service we can use: 
    service postgresql status 

    systemctl status postgresql 

One instance / installation of PostgreSQL can run multiple clusters od PostgreSQL.

to check the list of clusters running in the machine:
    pg_lsclusters

For this installation we have got one - 10-main. 
The cluster named running PostgreSQL 10. To check this cluster service status, we can use the command below.
    service postgresql@10-main status 

    systemctl status postgresql@10-main

### stop service
    systemctl stop postgresql 

    service postgresql stop

### start service
    systemctl start postgresql

    service postgresql start

### Restart service
    systemctl restart postgresql

### Reload service
    systemctl reload postgresql

### pg_lsclusters
show information about all PostgreSQL clusters

    pg_clusters

## Connect to postgresql 
First, the installations create a system user called postgres. First we need to change to that user:
    su - postgres 

With the postgres user we use the psql tool to connect: 
    psql -u 127.0.0.1

Set the password of the postgres user in postgres database to connect using another user, Since we cannot login without providing password, we set new password for user. : 
    \password 

To connect now from any user we can use:
    psql -U postgres -W -h 127.0.0.1

list all databases: 
    \l

list all usesr:
    \du

quit:
    \q

## Uninstall postgres 10
    sudo apt-get remove postgresql-10 \
    postgresql-client-10 \
    postgresql-client-common \
    postgresql-common

Remove postgresql directory: 

    rm -rf /var/lib/postgresql

Remove postgresql configuration directories:

    rm -rf /etc/postgresql 
    rm -rf /etc/postgresql-common

Reference:
https://www-howtodojo-com.translate.goog/install-postgresql-10-ubuntu-20-04/?_x_tr_sl=en&_x_tr_tl=es&_x_tr_hl=es&_x_tr_pto=sc






