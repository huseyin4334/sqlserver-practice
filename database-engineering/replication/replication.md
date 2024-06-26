# Replication

- Replication is the process of copying data from one database to another. 
- This process is used to improve the availability of data. In the event of a failure, the data can be recovered from the replicated database. 
- Replication can also be used to distribute data across multiple databases to improve performance.

## Demo

- We will create master and slave databases.
- We will configure the master database to replicate data to the slave database.

### Steps

1. Create a new databases named `master` and `slave`.

#### Create Databases
 
``` bash

# Create the master database.
docker run -d -p 5432:5432 --name master -e POSTGRES_PASSWORD=master_pass -e POSTGRES_DB=master -v /rep/master_data:/var/lib/postgresql/data postgres

# Create the slave database.
docker run -d -p 5433:5432 --name slave -e POSTGRES_PASSWORD=slave_pass -e POSTGRES_DB=slave -v /rep/slave_data:/var/lib/postgresql/data postgres

```

#### Configure Databases

1. Master Database

``` bash

cd /rep/master_data

# Edit the postgresql.conf file. This file contains the configuration settings for the database.
vim pg_hba.conf

# Add the following line to the file. This line allows the slave database to connect to the master database.
host replication slave all md5

:wq (in vim)
```

2. Slave Database

``` bash

cd /rep/slave_data

# Edit the postgresql.conf file. This file contains the configuration settings for the database.
vim postgresql.conf

# Find the following line in the file and uncomment it. This line enables replication.
# application_name is the name of the application that is connecting to the database.
primary_conninfo = 'application_name=slave1 host=<master_database_host> port=5432 user=slave password=slave_pass'

:wq (in vim)

# Create standby.signal file in the data directory. This file tells the database that it is a standby server.
touch standby.signal
```


``` bash

cd /rep/master_data

vim postgresql.conf

# Find the following line in the file and uncomment it. This means that the slave database can connect to the master database.
# I can write multiple slave databases separated by commas. 'any' means that any slave database can connect to the master database.
# 'first 2 (slave1, slave2, slave3)' means that the first two slave databases can connect to the master database.
sync_standby_names = 'first 1 (slave1)'
```

#### Start Databases

``` bash

docker stop master slave

docker start master slave

# We should see the following message in the logs.
# Master Database
# LOG:  database system is ready to accept connections
# LOG:  standby "slave1" is now a synchronous standby with priority 1

# Slave Database
# LOG:  database system is ready to accept read only connections
# LOG:  started streaming WAL from primary at 0/1000000 (timeline 1)

# "streaming WAL from primary" means WAL (Write Ahead Log) is being copied from the master database to the slave database.
# 0/1000000 is the position in the WAL file that is being copied.
# timeline 1 is the timeline of the WAL file.
```

## Pros and Cons
### Pros
- Improved availability of data.
- Horizontal scaling.
- Region based queries - DB per region.

### Cons
- Complexity. (Implementation, maintenance, monitoring, etc.)
- Data consistency.
- Slow writes.