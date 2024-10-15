# Select the MariaDB pod on NUC6 and go to command prompt:
mariadb -u root -p$MARIADB_ROOT_PASSWORD
flush tables with read lock;
show variables like 'gtid_binlog_pos';  

# Take results from above and set gtid_slave_pos for last step

# From another window on same cluster member:
mariadb-dump -h mariadb.mariadb.svc.cluster.local -u root -p$MARIADB_ROOT_PASSWORD -B gitea homeassist phpmyadmin ucdialplans vaultwarden > /bitnami/mariadb/data/mariadb-backup.sql

# Once done, then unlock from first:
unlock tables;

# From KenPC 
Copy backup from NUC6 to ONode1 /home/ubuntu/mariadb-repl-backup.sql

# From ONode1
```
mv /home/ubuntu/mariadb-backup.sql /var/mariadb/
```

# Then on Onode1 mariadb pod command prompt:
```
mariadb -u root -p$MARIADB_ROOT_PASSWORD
```
```
stop slave;
drop database gitea;
drop database homeassist;
drop database ucdialplans;
drop database vaultwarden;
drop database phpmyadmin;
```

# Exit to prompt and run
```
mariadb -u root -p$MARIADB_ROOT_PASSWORD < /bitnami/mariadb/mariadb-backup.sql
```

# Then run 
```
mariadb -u root -p$MARIADB_ROOT_PASSWORD
```
```
set global gtid_slave_pos = "0-1-19420";
change master to
    master_host='cloud-egress',
    master_user='replicator',
    master_password='***REMOVED***',
    master_port=3306,
    master_connect_retry=10,
    master_use_gtid=slave_pos;

start slave;
```

# Check status
```
show slave status;
```

# If you get replication errors, try skipping the error and continuing:
```
STOP SLAVE;
SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
START SLAVE;
SELECT SLEEP(5);
SHOW SLAVE STATUS;
```