# mysqlchk

Based on ClusterControl's mysqlchk script but built for Galera. It detects the Galera status on the database node as per below:

* if wsrep_cluster_status is Primary,
 * return 'MySQL master is running.'

* else
 * return 'MySQL is \*down\*'

# How to use

1) Install HAProxy on the selected node. 

2) Grab the file from this repo and place it inb /usr/local/sbin/

3) Update required informations :
```bash
MYSQL_HOST="localhost"
MYSQL_PORT="3306"
MYSQL_USERNAME='user'
MYSQL_PASSWORD='password'
```
4) You are good to go. Now ensure in HAProxy you have 2 listeners (3307 for write, 3308 for read) and use ``option tcp-check`` & ``tcp-check expect`` to distinguish master/slave similar to example below:
```bash

listen  galera
        bind *:3306
        mode tcp
        #timeout client  10800s
        #timeout server  10800s
        balance leastconn
        option tcp-check
        tcp-check expect string is\ running.
        default-server port 9200 inter 2s downinter 5s rise 3 fall 2 # slowstart 60s maxconn 64 maxqueue 128 weight 100
        server node1 192.168.55.111:3306 check
        server node2 192.168.55.112:3306 check
        server node3 192.168.55.113:3306 check
```
