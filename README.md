# mysqlchk

Based on ClusterControl's mysqlchk script but built for Galera. It detects the Galera status on the database node as per below:

* if wsrep_cluster_status is Primary,
 * return 'MySQL master is running.'

* else
 * return 'MySQL is \*down\*'

# How to use

1) Install HAProxy on the your selected node. ClusterControl will install the default mysqlchk that need to be replaced with this one on each of the database node. By default, it's located under ``/usr/local/sbin/mysqlchk``

2) Grab the file from this repo and replace it.

3) Update require information (``SLAVE_LAG_LIMIT`` is in seconds):
```bash
MYSQL_HOST="localhost"
MYSQL_PORT="3306"
MYSQL_USERNAME='root'
MYSQL_PASSWORD='password'
```
4) You are good to go. Now ensure in HAProxy you have 2 listeners (3307 for write, 3308 for read) and use ``option tcp-check`` & ``tcp-check expect`` to distinguish master/slave similar to example below:
```bash

listen  haproxy_192.168.55.110_3308_read
        bind *:3308
        mode tcp
        timeout client  10800s
        timeout server  10800s
        balance leastconn
        option tcp-check
        tcp-check expect string is\ running.
        default-server port 9200 inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 64 maxqueue 128 weight 100
        server 192.168.55.111 192.168.55.111:3306 check
        server 192.168.55.112 192.168.55.112:3306 check
        server 192.168.55.113 192.168.55.113:3306 check
```
