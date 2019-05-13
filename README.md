# proxysql

### Start a docker:

```
systemctl start docker 
```


```
docker stop $(docker ps -a -q)
docker container rm $(docker container ls -aq)


docker pull actency/docker-mysql-replication:5.7
rm -rf data/mysql-*/*
mkdir data/mysql-master
mkdir data/mysql-slave

docker run -d \
 --name mysql_master \
 -v $PWD/data/mysql-master:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=mysqlroot \
 -e MYSQL_USER=example_user \
 -e MYSQL_PASSWORD=mysqlpwd \
 -e MYSQL_DATABASE=example \
 -e REPLICATION_USER=replication_user \
 -e REPLICATION_PASSWORD=myreplpassword \
 -p 3306:3306 \
 actency/docker-mysql-replication:5.7

docker exec -it mysql_master mysql -uroot -pmysqlroot -e "SHOW MASTER STATUS\G;"
docker exec -it mysql_master mysql -uroot -pmysqlroot -e "show variables where variable_name like 'read_only';"

docker run -d \
 --name mysql_slave \
 -v $PWD/data/mysql-slave:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=mysqlroot \
 -e MYSQL_USER=example_user \
 -e MYSQL_PASSWORD=mysqlpwd \
 -e MYSQL_DATABASE=example \
 -e REPLICATION_USER=replication_user \
 -e REPLICATION_PASSWORD=myreplpassword \
 --link mysql_master:master \
 -p 3307:3306 \
 actency/docker-mysql-replication:5.7


 docker exec -it mysql_slave mysql -uroot -pmysqlroot -e "SHOW SLAVE STATUS\G;"
 docker exec -it mysql_slave mysql -uroot -pmysqlroot -e "SET GLOBAL read_only=true;"
docker exec -it mysql_slave mysql -uroot -pmysqlroot -e "show variables where variable_name like 'read_only';"
```

STOP SLAVE;
SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1; 
START SLAVE;


## Monitoring:


```
docker pull grafana/grafana

mkdir grafana 
ID=$(id -u) 
docker run -d --name grafana \
              --user $ID \
              --volume "$PWD/grafana:/var/lib/grafana" \
              -p 3000:3000 \
              --link mysql_master:master \
              --link mysql_slave:slave \
              grafana/grafana
```


mysql -u admin -padmin -h 127.0.0.1 -P6032 --prompt='Admin> '
load mysql servers to runtime

mysql -u example_user -pmysqlpwd -h 127.0.0.1 -P6033 --prompt='ProxySQL> '
mysql -u example_user -pmysqlpwd -h 192.168.1.5 -P3306 --prompt='Local> '


restart:
/etc/init.d/proxysql stop
rm -rf /var/lib/proxysql/*
scp arodrigues@192.168.1.5:/home/arodrigues/projetos/proxysql/proxysql/etc/proxysql.cnf /etc/proxysql.cnf
/etc/init.d/proxysql start

scp /etc/proxysql.cnf arodrigues@192.168.1.5:/home/arodrigues/projetos/proxysql/proxysql/etc/proxysql.cnf 


### config
```
mysql -u admin -padmin -h 127.0.0.1 -P6032 --prompt='Admin> '


 SELECT * FROM mysql_servers;
 SELECT * from mysql_replication_hostgroups;
 SELECT * from mysql_query_rules;

INSERT INTO mysql_servers(hostgroup_id,hostname,port) VALUES (20,'192.168.1.5',3307);
INSERT INTO mysql_servers(hostgroup_id,hostname,port) VALUES (10,'192.168.1.5',3306);
SELECT * FROM mysql_servers;

select * from global_variables where variable_name like 'mysql-monitor%';
UPDATE global_variables SET variable_value='2000' WHERE variable_name IN ('mysql-monitor_connect_interval','mysql-monitor_ping_interval','mysql-monitor_read_only_interval');

LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;

SELECT * FROM monitor.mysql_server_connect_log ORDER BY time_start_us DESC LIMIT 10;
SELECT * FROM monitor.mysql_server_ping_log ORDER BY time_start_us DESC LIMIT 10;


SELECT * FROM monitor.mysql_server_read_only_log ORDER BY time_start_us DESC LIMIT 10;

INSERT INTO mysql_users(username,password,default_hostgroup) VALUES ('example_user','mysqlpwd',10);
update mysql_users set password='mysqlpwd';

LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
 mysql -u example_user -pmysqlpwd -h 127.0.0.1 -P6033 -e "SELECT 1"
 mysql -u example_user -pmysqlpwd -h 127.0.0.1 -P6033 -e "SELECT @@port"

```

### docker
```

docker build -t afonsoaugusto/proxysql:latest  .
docker container rm proxysql -f 
docker run -d --name proxysql \
              -p 6033:6033 \
              -p 6032:6032 \
              -p 6080:6080 \
              -v $PWD/etc/proxysql.cnf:/etc/proxysql.cnf \
              --link mysql_master:master \
              --link mysql_slave:slave \
              afonsoaugusto/proxysql


docker logs proxysql
docker exec -it proxysql /etc/initi.d/proxysql restart

```


Reference:
https://hub.docker.com/r/actency/docker-mysql-replication/
https://github.com/sysown/proxysql