# wal archiving (continuous archiving) and point in time recovery

### Wals  
### pg_wals enables point in time recovery 
#### Wal files - files in which the trasnactions are recorded by postgres before be recorded into the data files. This enable to recovery agains crash evens to a stable state seconds before the crash events occurs. This is possible by the wal files. They are physics and rotative files, in which postgres writes the operations before record it into the data files. The wal files are generated in operations (hot backups).

#### the wals are stored in the data directory, /var/lib/postgresql/12/main/pg_wal for postgres >= 10
#### for versions of postgres < 10 /var/lib/postgres/10/main/pg_xlog

### To enable point in time recovery we need: 
#### 1 - Base backup
#### 2 - The wal files that record the transaction from the moment of the last base backup until seconds before the crash event. 

#### Wal archiving 
#### Is the process of copy of the wal files into a local or remote location with the process to store it into safe place a crash events occurs and get that files to restore the database into a consistent state

#### Continuos backup - This is called like that because with the wal archiving we are making a backup of the transactions that occurs and are recorded into the the wal files, we are making a backup of the wal files in a local/remote location, (remember that the wal files are rotative, are reused), enable us to recovery the data to any point of time. To that we need:
#### The wals - from a base_backup enables rebuilt the trasactions commited in the time line until moments before a crash event. 
#### To recovery agains a crash event:
##### 1 - Restore the base backup
##### 2 - With the service stoped we copy the wal files into the wal directory
##### 3 - Recovery file 
##### With this we enable postgres to restore the database to the last consistent state
##### Also, we create a new base backup and that's create a new time line in which we keep with the process of wal archiving to be able to restore the database if a new crash event occurs.

### To enable wal archiving we need to modify the postgres config file, we have to set the parameters and we can also some additional paramets:
#### 1 - wal_archiving= this parameter we can have minimal, replica  - by default can be minimal or replica - depends of the version - To enable wal archiving we need to set it to replica
#### 2 - archive_mode=on - by default is set to off
#### 3 - archive_command= - by default is disable - this command is a bash command that postgres use to copy/move the wal files into a location that we set to store the wal files, a local/remote location. 
#### We can have others parameters like wal_size (min_wal_size, max_wal_size), max_wal_senders (postgres recommend 3), archive_timeout

#### to locate the config file we can:
psql -c "show config_file;"
pg_lsclustesr

#### example 
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /mnt/backups/wal_archive/%f && cp %p /mnt/backups/wal_archive/%f'
max_wal_senders = 1

#### restart postgres
systemctl restart postgresql
ps -ef | grep postgres

#### check the configuration
psql -c "show wal_level;"
psql -c "show archive_mode;"
psql -c "show archive_command;"
psql -c "show wal_senders;"


## How to use the base backups for point in time recovery
### To create a base backcup use the tool - pg_basebackup
    pg_basebackup [options]

#### important options
    -D -- Specify the directory in which the file will be stored. 
    -F -- format p|t - plain text or tar
    -r -- Define a max tranfer rate - default 32kb to 1024 MB 
    -R - Refers to the recovery.conf file - with this options creates the file.
    -T -- Table space mapping - to map the original tablespace of the database to new ones to be ready for a new implementation or a new instance of the database.
    -X -- set the wal method none|fetch|stream - none do not create a backup of the wal files, fetch create a copy at the end of the backup, stream - creates a new connection to create a backup of the wal files in paralel with the backup of the data files.
    -z -- to compres the output into a zip (the tar output)
    -Z -- to compress to specific compression rate.

    General Options:
    -c - checkpoints - to force checkpints - fast option
    -C - create replication solet when is a publisher or primary database.
    -l - label - create a label for the backcup file
    -P - to check the progress
    --no-verify-checksums - do not perform checks of the data.

    Connection:
    -d dabasename
    -h host
    -p port
    -s interval status - if set to 0 will do not send status packages
    -U username
    -w no password
    -W password

### to crate a base backup in tar format, compressed into a zip file and into a specific directory and also that creates the recovery.conf file:
    pg_basebackup -D /mnt/backups/base_backups -Ft -R -z -P

#### Remember that the specific location in which the backup file will be stored the postgres users has the rights to create files in the location.

#### How to create a recovery point time
##### 1 create a base backup - create base backup with checkpoint fast to do it fast and wal-method none to do not backup the wal files
    pg_basebackup -D /home/mnt/backups/base_backup -Ft -P -R -z --checkpoint=fast --wal-method=none

##### 2 check backup files - tablespace files
    select oid, spcname, pg_tablespace_location(oid) from pg_tablespace;


16396 | tbs_recetas | /var/lib/postgresql/12/tbs_recetas
 16397 | tbs_lab     | /var/lib/postgresql/12/tbs_lab

###### the backup files of the table space use the oid of the table space. So, the query above shows the oid of the table space. 
    16396.tar.gz  -- table space backup tbs_recetas
    16397.tar.gz  -- table space backup tbs_lab
    base.tar.gz   -- pg_data backup 

##### 3 create the point in time - use a postgres function in sql and requires a name:
    select pg_create_restore_point('mypoint');

##### 4 to test here, we drop two databases, once we have the required to restore them:
    drop database recetas;
    drop databse labboratorio;

##### 5 stops the postgres service
    systemctl stop postgres

##### 6 clean the main and tablespace files - to do not remove this files mv 
    mv /var/lib/postgres/12/main/* /mnt/backups/olds/main_old
    mv /var/lib/postgres/12/tb_recetas/* /mnt/backups/olds/tbs_recetas_old
    mv /var/lib/postgres/12/tb_lab/* /mnt/backups/olds/tbs_labs_old

#### 7 move the backups files into the corresponding files
    cp /mnt/backups/basebackup/base.tar.gz /var/lib/postgres/12/main/
    cp /mnt/backups/basebackup/16396.tar.gz /var/lib/postgres/12/tbs_recetas
    cp /mnt/backups/basebackup/16397.tar.gz /var/lib/postgres/12/tbs_lab

#### 8 unzip the files into their corresponding directories
#### 9 set the recovery.conf file - for postgres >= 12 their is not a recovery.conf files, this only exists < 12. But, that file indicates to postgres when it start, start in recovery mode and set the recovery point, to a which point postgres will restore the trasnactions based on the wal files. So, for postgres >= 12 there are two files that in our cases were generated automatically by the basebackup (option -R), standby.signal and the postgresql.auto.conf. 
##### The standby.signal file is a empty file, that when exists indicates to postgres when it start, start into recovery mode.
##### The postgresql.auto.conf set the parameters to indicate the point in time to restore
#### for this we have to add two parameters, one the command that postgres will use to get the wal files and the point time in which the database will be restored, recovery point. Also instead of recovery_target_name could be recovery_target_time = date - to indicates the point 

echo "restore_command = 'cp /mnt/backups/wal_archive/%f %p'" >> postgresql.auto.conf
echo "recovery_target_name = 'pointime1'" >> postgresql.auto.conf

#### the recovery target name referes to a pre-stablish point in time. There is other use-case when a event occurs and we have to restore to a not pre-stablish point in time. So, with the above we recovery to the point in time that we stablish. 

#### 10 - Start the databse engine - Start in recovery and applied the transactions from the wal files until the point that we stablish.
    systemctl start postgresql

#### 11 - Start in recovery - only accept read connections. Once postgres restore, we have to execute:
    psql -p 5433 -c "select pg_wal_replay_resume();"
#### from here, we are with the databse in the state of the recovery point that we specify. Creates a new timeline, a timeline in which we avoid drop the two database that we drop before. So, in the wal files we can find a file that speficy that we are now in a new timeline
00000002.history






    
