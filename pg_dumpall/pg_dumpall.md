# USING PG_DUMPALL TO CREATE BACKUPS FOR POSTGRESQL 

pg_dumpall is a tool that allows you to create a logical backup in sql format for all the postgresql cluster. 
Underground pg_dumpall use pg_dump for any database in the cluster.


## Help
    pg_dumpall --help 

## Usage
    pg_dumall [options]

## create a backup 
If we use only pg_dumpall we create all the sql instructions of all the database of the cluster 

    pg_dumpall 

Only using this command without any Instruction shows all the sql instructions in the shell, for that, we need to redirect that output into a file.

Vuelca la salida por pantalla - Dump

The format of the backup file, can have any extension, but the recommendation is the sql extension.

    pg_dumpall > backup1.sql

The sequence of pg_dumpall:
Global objects, like general settings like encoding, roles, tablespaces and then the databases one by one 

The dumps in postgres are smart, the sql instrucction that generates follow an smart logic order, for example, fist the global objects, later the database, parameters, schemas, tables, copy data, creates constrains 
pg_dumpall, los sql dumps son inteligentes, las instrucciones son secuenciales siguiendo un orden logico,
primero por ejemplo crea las tablas, luego copia los datos, luego las primary key y postgeriormente las foraneas
y asi sucesivamente. 

## GENERAL OPTIONS
    -f - To references the specific file where the output will be stored 
    -V - To print the pg_dumpall version used 
    -v - verbose - Shows in a detailed form what pg_dumpall is doing 


Generates a backup in the specific file/location: 
    pg_dumpall -p 5433 -W -U postgres -f /home/sortiz/logical_backups/dumpall.sql

The options --lock-wait-timeout=timeout, the timeout is an integer that represent time in miliseconds, references to the time that pg_dumpall will wait for a table to be unlocked. In postgresql, pg_dumpall is bloqued by DDL instructions, like Alter tables instructions, if we set this parameter, if the pg_dumpall wait the timie specified will done with an error message. Wait for table blocks. By default, pg_dumpall wait until the tables are unlocked

THe option --no-sync indicates to pg_dumpall to do not wait for checkpoints, which means, do no wait until all the files are written in a secure way in the disk. With that option, pg_dumpall do not wait for that. This is not recommended, no garantiza un backup valido. Por lo cual no es recomendado para sistemas de produccion.

## Options to control the output content 
### -a - data only

### -c - Clean - Clean all the objects and then recreate them. Generates a sql script file with a logic order of instructions that first drop the objects, drop database, drop tablespace, drop roles and then recreates them, first roles, then tablespaces and the database and all their objects (tables, constrains, indexes, data...)

## to generate a backup with drop objects intructions - also add the if exists - drop if the object exists
## Then recreate the objects  and the data
## -c - drop the objects
pg_dumpall -p 5433 -w -U postgres -f $workdir/pgdumpall_clean.sql -c --if-exists -v


## -s - schema only -> backup for the schema / include only the database structure
pg_dumpall -p 5433 -w -U postgres -f $workdir/pgdumpall_schema.sql -s

## -a - this option create a backup with the data only
pg_dumpall -p 5433 -w -U postgres -f $workdir/pgdumpall_data.sql -a -v

## -g - Only backup de global objects -> in this case only creates a backup of tablespaces and roles
pg_dumpall -p 5433 -w -U postgres -f $workdir/pgdumpall_global.sql -g  -c

## -r - roles only -> creates a backup only for the database roles
pg_dumpall -p 5433 -w -U postgres -f $workdir/pgdumpall_roles.sql -r -c -v

## -t -> table spaces only
pg_dumpall -p 5433 -w -U postgres -f $workdir/pgdumpall_tbs.sql  -t -c -v

## --no-tablespaces -> backup all the cluster without tables spaces
pg_dumpall -p 5433 -w -U postgres -v -c --no-tablespaces -f $workdir/pgdumpall_no_tbs.sql

## -O no owner - skip restauration of object ownership - but create a backup of the ownership property for global objects
    pg_dumpall -p 5433 -w -U postgres -v -O -f -f $workdir/pgdumpall_no_owners.sql

## --no-comments - do not dump comments 
    pg_dumpall -p 5433 -w -U postgres -v -O -f -f $workdir/pgdumpall_no_owners.sql





