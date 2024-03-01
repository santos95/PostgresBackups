# POSTGRES COMMANDS 

## connect to the database to a specific cluster into the running postgres
    psql -p 5433

## change password to the postgres user:
    \password 

## List all databases
    \l

## List all users
    \du

## Select or use a database
    \c database

## Get Connection information
    \conninfo

## show schemas of the current database
    \dn 

    List of schemas
    Name   |    Owner
    ---------+-------------
    public  | postgres
    sch_lab | laboratorio

## List all tablas into an schema
    \dt sch_lab.*

    # to get more informacion
    \dt+ sch_lab 

## List tablaespaces
    \db
### List tablesspaces with more information - size and access privileges
    \db+

## clear 
    \! clear

    
