# oracle-container-scripts

## Pre-reqs
- Delphix environment should have an OS user with `uid=54321` and `gid=54321` (same as the Oracle user inside the container)
- Set this user to the environment default
- create a dSource in Delphix named `oracle-container-push` (example mount path: `/mnt/provision/oracle-container-push`)
- copy the shell scripts contained in this repository into `oracle-container-push` mount point path take a snap

## Docker run do duplicate the database
 
```
export ORACLE_SID={instance and database names inside de container}
export ORACLE_PWD={sys user password in the source database}
export CONTAINER_DB={false for non-CDB, true for multi-tenant}
export PRIMARY_DB_CONN_STR={EZ connect string for source database. It's best to use the db unique name (db_name + db_domain) after the slash, if it's registered in the listener. Ex: 10.160.1.21:1521/orasrc1.delphix.lab}

docker run -d --name staging-oracle-database --memory 2G \
-p 1521:1521 -p 5500:5500 -p 2484:2484 \
--ulimit nofile=1024:65536 --ulimit nproc=2047:16384 --ulimit stack=10485760:33554432 --ulimit memlock=3221225472 \
-e ORACLE_SID=${ORACLE_SID} \
-e ORACLE_PWD=${ORACLE_PWD} \
-e INIT_SGA_SIZE=1500 \
-e INIT_PGA_SIZE=500 \
-e ORACLE_EDITION=enterprise \
-e ORACLE_CHARACTERSET=AL32UTF8 \
-e ENABLE_ARCHIVELOG=true \
-e ENABLE_TCPS=true \
-v /mnt/provision/oracle-container-push:/opt/oracle/oradata \
-e CLONE_DB=true \
-e PRIMARY_DB_CONN_STR="'${PRIMARY_DB_CONN_STR}'" \
-e DATAFILE_DESTINATION='/opt/oracle/oradata/${ORACLE_SID}/' \
-e RECOVERY_AREA_DESTINATION='/opt/oracle/oradata/${ORACLE_SID}/fast_recovery_area/onlinelog/' \
-e CONTAINER_DB=${CONTAINER_DB} \
--entrypoint /opt/oracle/oradata/runOracle.sh \
533693045312.dkr.ecr.us-west-2.amazonaws.com/oracle-database:19.3.0-ee
```
