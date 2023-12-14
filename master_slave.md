


```
 #!/bin/bash

# Create a Podman network named rahulnetwork with subnet 172.19.0.0/24
echo "Creating Podman Network: rahulnetwork"
podman network create --subnet=172.19.0.0/24 rahulnetwork

# Create directories for mount volumes
echo "Creating Directories for Mount Volumes"
mkdir -p /home/rahul/data/psql/master
mkdir -p /home/rahul/data/psql/slave
mkdir -p /home/rahul/data/psql/repl

# Run Master Container
echo "Running Master Container"
podman run -d \
  --network rahulnetwork --ip 172.19.0.101 -p 5432:5432 \
  --name master -h master \
  -e "POSTGRES_DB=postgres" \
  -e "POSTGRES_USER=postgres" \
  -e "POSTGRES_PASSWORD=redhat" \
  -v /home/rahul/data/psql/master:/var/lib/postgresql/data \
  docker.io/postgres

# Wait for the Master container to be ready
echo "Waiting for Master container to be ready"
until podman exec master pg_isready -q; do
  sleep 1
done

# Run Slave Container
echo "Running Slave Container"
podman run -d \
  --network rahulnetwork --ip 172.19.0.102 -p 5433:5432 \
  --name slave -h slave \
  -e "POSTGRES_DB=postgres" \
  -e "POSTGRES_USER=postgres" \
  -e "POSTGRES_PASSWORD=redhat" \
  -v /home/rahul/data/psql/slave:/var/lib/postgresql/data \
  -v /home/rahul/data/psql/repl:/var/lib/postgresql/repl \
  docker.io/postgres

# Wait for the Slave container to be ready
echo "Waiting for Slave container to be ready"
until podman exec slave pg_isready -q; do
  sleep 1
done

echo "Containers are up and running!"

```

```
# Run the script 
sh -x post.sh 

```


```
Error

WARN[0000] Error validating CNI config file /etc/cni/net.d/rahulnetwork.conflist: [plugin bridge does not support config version "1.0.0" plugin portmap does not support config version "1.0.0" plugin firewall does not support config version "1.0.0" plugin tuning does not support config version "1.0.0"] 

Soluation:

cd /etc/cni/net.d/

vim rahulnetwork.conflist

change the (version 1.0.0 => 0.4.0)

save it

```


```
# Now, let's check the networks

podman network ls

# If you want to delete the networks

podman network rm -f rahulnetwork
```

![](/1.png)


```
# Check the status of podman contanior.
podman ps

# Access the master postgres
podman exec -it master bash

# Connect the database
psql -U postgres

# Create the role 
CREATE ROLE rahul WITH LOGIN REPLICATION CONNECTION LIMIT 5 PASSWORD 'redhat';

# Check the role status
\du

# Exit 
\q

# Path for postgresql.confi
cd /var/lib/postgresql/data/

# Update the contanior
apt updare

# Install the vim inside the contanior
apt install vim

```


```
# Now, Setup host inside the pg_hba.conf file
echo "host replication rahul 172.19.0.102/24 md5" >> pg_hba.conf

# Path for pg_hba.cong
vim /var/lib/postgresql/data/pg_hba.conf


# Configure the postgresql.conf file
vim postgresql.conf


archive_mode = on	
archive_command = '/bin/date'
max_wal_senders = 10
wal_keep_size = 16	
synchronous_standby_names = '*'


# Now, Restart the master postgres
podman restart master

# check the status
podman ps

```




```
# Now, Access the slave postgres
podman exec -it slave bash

# Backup all the data
pg_basebackup -R -D /var/lib/postgresql/repl -Fp -Xs -v -P -h 172.19.0.101 -p 5432 -U rahul

# Exit from contanior
Ctrl+d

# Remove the slave postgres contanior
podman rm -f slave

# Chcek the status
podman ps


# Now, path of backup file here
cd data/psql/

# check the files here
ls

# Now, Remove the slave diretory 
rm -rf slave

# Check the Removed slave from here
ls

# Rename the repl to slave
mv repl slave

# Now, Check the slave file is here
ls


```


```
# Now, Setup slave podman contonire 
podman run -d --network rahulnetwork --ip 172.19.0.102 -p 5433:5432 --name slave -h slave -e "POSTGRES_DB=postgres" -e "POSTGRES_USER=postgres" -e "POSTGRES_PASSWORD=redhat" -v /home/ldap/data/psql/slave:/var/lib/postgresql/data docker.io/postgres

# Check the slave podman 
podman ps


```


```
# Now Access the master postgres 
podman exec -it master bash

# Connect the postgresql database
psql -U postgres


# Check the status of replication on master 
select * from pg_stat_replication;

```
![](/3.png)



```
# Now Create the database in master
CREATE DATABASE rahul;

# Exit
\q

exit

```



```
# Now, check the database rahul inside the slave
podman exec -it slave bash

# Connect the database
psql -U postgres

# check the rahul database
\l
 
```

![](/4.png)
