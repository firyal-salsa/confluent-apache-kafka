# Confluent Apache Kafka 
Dari proses instalasi, memasang SSL, SASL, sampai mengakses suatu topic dengan ACLs


## Install

Menggunakan Ubuntu 
```bash
sudo nano /etc/apt/sources.list.d/archive_uri-https_packages_confluent_io_clients_deb-noble.list
```
ubah noble menjadi focal
```bash
sudo apt-get update && \
sudo apt-get install confluent-platform && \
sudo apt-get install confluent-security
```


Menggunakan RHEL / CentOs / Rocky / Amazon Linux
```bash
sudo yum clean all && \
sudo yum install confluent-platform && \
sudo yum install confluent-security
```


## Memasang SSL 

Pastikan di file /etc/host/ sudah terdefinisi setiap node beserta toolsnya : <br>
<nama_ip> node1.zookeeper node1.kafka node1.schema node1.connect node1.ksql node1.rest <br>
<nama_ip> node2.zookeeper node2.kafka node2.schema node2.connect node2.ksql node2.rest node2.cc <br>
<nama_ip> node3.zookeeper node3.kafka node3.schema node3.connect node3.ksql node3.rest <br>

```bash
sudo openssl req -new -x509 -keyout ca-root.key -out ca-root.crt -days 365 -subj '/CN=server/OU=FS/O=ADI/L=JAKBA
R/ST=DKI/C=ID' -passin pass:confluent -passout pass:confluent
```
```bash

```
```bash
sudo keytool -genkey -noprompt -alias server -dname "CN=server,OU=FS,O=ADI,L=JAKBAR,S=DKI,C=ID" -ext "SAN=dns:no
de1,dns:node2,dns:node3,dns:localhost,dns:node1.zookeeper,dns:node1.kafka,dns:node1.schema,dns:node1.connect,dns:node1.
ksql,dns:node1.rest,dns:node2.zookeeper,dns:node2.kafka,dns:node2.schema,dns:node2.connect,dns:node2.ksql,dns:node2.res
t,dns:node2.cc,dns:node3.zookeeper,dns:node3.kafka,dns:node3.schema,dns:node3.connect,dns:node3.ksql,dns:node3.rest" -k
eystore kafka.server.keystore.jks -keyalg RSA -storepass confluent -keypass confluent -storetype pkcs12
```
```bash
sudo keytool -keystore kafka.server.keystore.jks -alias server -certreq -file server.csr -storepass confluent -k
eypass confluent -ext "SAN=dns:node1,dns:node2,dns:node3,dns:localhost,dns:node1.zookeeper,dns:node1.kafka,dns:node1.sc
hema,dns:node1.connect,dns:node1.ksql,dns:node1.rest,dns:node2.zookeeper,dns:node2.kafka,dns:node2.schema,dns:node2.con
nect,dns:node2.ksql,dns:node2.rest,dns:node2.cc,dns:node3.zookeeper,dns:node3.kafka,dns:node3.schema,dns:node3.connect,
dns:node3.ksql,dns:node3.rest"
```
Buat file server.cnf 
```bash
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
CN = server
[v3_req]
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = node1.zookeeper
DNS.2 = node1.kafka
DNS.3 = node1.schema
DNS.4 = node1.connect
DNS.5 = node1.ksql
DNS.6 = node1.rest
DNS.7 = node2.zookeeper
DNS.8 = node2.kafka
DNS.9 = node2.schema
DNS.10 = node2.connect
DNS.11 = node2.ksql
DNS.12 = node2.rest
DNS.13 = node2.cc
DNS.14 = node3.zookeeper
DNS.15 = node3.kafka
DNS.16 = node3.schema
DNS.17 = node3.connect
DNS.18 = node3.ksql
DNS.19 = node3.rest
```
```bash
sudo openssl x509 -req -CA ca-root.crt -CAkey ca-root.key -in server.csr -out server-signed.crt -sha256 -days 365 -CAcreateserial -passin pass:confluent -extensions v3_req -extfile server.cnf
```
```bash
sudo keytool -noprompt -keystore server.keystore.jks -alias caroot -import -file ca-root.crt -storepass confluent -keypass confluent
```
Menggabungkan kedua file tersebut ke full-chain.xrt
```bash
sudo sh -c 'cat server-signed.crt ca-root.crt > full-chain.crt'
```
```bash
sudo keytool -noprompt -keystore kafka.server.keystore.jks -alias server -import -file full-chain.crt -storepass confluent -keypass confluent
```
```bash
 sudo keytool -noprompt -keystore kafka.server.truststore.jks -alias caroot -import -file ca-root.crt -storepass 
confluent -keypass confluent
```

```bash
scp -r confluent@<alamat_ip>:/var/ssl/private /var/ssl/private
```

### Zookeeper
```bash
sudo nano /etc/kafka/zookeeper.properties
```

Edit file /etc/kafka/zookeeper.properties
```bash
4lw.commands.whitelist=*
admin.enableServer=false
authProvider.x509=org.apache.zookeeper.server.auth.X509AuthenticationProvider
autopurge.purgeInterval=1
autopurge.snapRetainCount=10
dataDir=/data1/zookeeper/
initLimit=5
maxClientCnxns=0
secureClientPort=2182
server.1=node1.zookeeper:2888:3888
server.2=node2.zookeeper:2888:3888
server.3=node3.zookeeper:2888:3888
serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
ssl.clientAuth=none
ssl.keyStore.location=/var/ssl/private/kafka.server.keystore.jks
ssl.keyStore.password=confluent
ssl.trustStore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.trustStore.password=confluent
syncLimit=2
```

Buat file /data1/zookeeper dengan akses sebagai berikut
```bash
sudo chown -R cp-kafka:confluent /data1/kafka
```

Tambahkan file berikut yang berisi nomor dari node sekarang
```bash
touch /data/zookeeper/myid
```

```bash
systemctl daemon-reload
```
```bash
systemctl start confluent-zookeeper.service
```

### Broker

Edit /etc/kafka/server.properties 
```bash
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=2

############################# Socket Server Settings #############################

# The address the socket server listens on. If not configured, the host name will be equal to the value of
# java.net.InetAddress.getCanonicalHostName(), with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=SSL://:9092

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
advertised.listeners=SSL://node3.kafka:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
listener.security.protocol.map=SSL:SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/data1/kafka

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
#log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=node1.zookeeper:2182,node2.zookeeper:2182,node3.zookeeper:2182

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000

##################### Confluent Metrics Reporter #######################
# Confluent Control Center and Confluent Auto Data Balancer integration
#
# Uncomment the following lines to publish monitoring data for
# Confluent Control Center and Confluent Auto Data Balancer
# If you are using a dedicated metrics cluster, also adjust the settings
# to point to your metrics kakfa cluster.
#metric.reporters=io.confluent.metrics.reporter.ConfluentMetricsReporter
#confluent.metrics.reporter.bootstrap.servers=localhost:9092
#
# Uncomment the following line if the metrics cluster has a single broker
#confluent.metrics.reporter.topic.replicas=1

############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0


############################# Confluent Authorizer Settings  #############################

# Uncomment to enable Confluent Authorizer with support for ACLs, LDAP groups and RBAC
#authorizer.class.name=io.confluent.kafka.security.authorizer.ConfluentServerAuthorizer
# Semi-colon separated list of super users in the format <principalType>:<principalName>
#super.users=
# Specify a valid Confluent license. By default free-tier license will be used
#confluent.license=
# Replication factor for the topic used for licensing. Default is 3.
confluent.license.topic.replication.factor=1

# Uncomment the following lines and specify values where required to enable CONFLUENT provider for RBAC and centralized ACLs
# Enable CONFLUENT provider 
#confluent.authorizer.access.rule.providers=ZK_ACL,CONFLUENT
# Bootstrap servers for RBAC metadata. Must be provided if this broker is not in the metadata cluster
#confluent.metadata.bootstrap.servers=PLAINTEXT://127.0.0.1:9092
# Replication factor for the metadata topic used for authorization. Default is 3.
confluent.metadata.topic.replication.factor=1

# Replication factor for the topic used for audit logs. Default is 3.
confluent.security.event.logger.exporter.kafka.topic.replicas=1

# Listeners for metadata server
#confluent.metadata.server.listeners=http://0.0.0.0:8090
# Advertised listeners for metadata server
#confluent.metadata.server.advertised.listeners=http://127.0.0.1:8090

############################# Confluent Data Balancer Settings  #############################

# The Confluent Data Balancer is used to measure the load across the Kafka cluster and move data
# around as necessary. Comment out this line to disable the Data Balancer.
confluent.balancer.enable=true

# By default, the Data Balancer will only move data when an empty broker (one with no partitions on it)
# is added to the cluster or a broker failure is detected. Comment out this line to allow the Data
# Balancer to balance load across the cluster whenever an imbalance is detected.
#confluent.balancer.heal.uneven.load.trigger=ANY_UNEVEN_LOAD

# The default time to declare a broker permanently failed is 1 hour (3600000 ms).
# Uncomment this line to turn off broker failure detection, or adjust the threshold
# to change the duration before a broker is declared failed.
#confluent.balancer.heal.broker.failure.threshold.ms=-1

# Edit and uncomment the following line to limit the network bandwidth used by data balancing operations.
# This value is in bytes/sec/broker. The default is 10MB/sec.
#confluent.balancer.throttle.bytes.per.second=10485760

# Capacity Limits -- when set to positive values, the Data Balancer will attempt to keep
# resource usage per-broker below these limits.
# Edit and uncomment this line to limit the maximum number of replicas per broker. Default is unlimited.
#confluent.balancer.max.replicas=10000

# Edit and uncomment this line to limit what fraction of the log disk (0-1.0) is used before rebalancing.
# The default (below) is 85% of the log disk.
#confluent.balancer.disk.max.load=0.85

# Edit and uncomment these lines to define a maximum network capacity per broker, in bytes per
# second. The Data Balancer will attempt to ensure that brokers are using less than this amount
# of network bandwidth when rebalancing.
# Here, 10MB/s. The default is unlimited capacity.
#confluent.balancer.network.in.max.bytes.per.second=10485760
#confluent.balancer.network.out.max.bytes.per.second=10485760

# Edit and uncomment this line to identify specific topics that should not be moved by the data balancer.
# Removal operations always move topics regardless of this setting.
#confluent.balancer.exclude.topic.names=

# Edit and uncomment this line to identify topic prefixes that should not be moved by the data balancer.
# (For example, a "confluent.balancer" prefix will match all of "confluent.balancer.a", "confluent.balancer.b",
# "confluent.balancer.c", and so on.)
# Removal operations always move topics regardless of this setting.
#confluent.balancer.exclude.topic.prefixes=

# The replication factor for the topics the Data Balancer uses to store internal state.
# For anything other than development testing, a value greater than 1 is recommended to ensure availability.
# The default value is 3.
confluent.balancer.topic.replication.factor=1

################################## Confluent Telemetry Settings  ##################################

# To start using Telemetry, first generate a Confluent Cloud API key/secret. This can be done with
# instructions at https://docs.confluent.io/current/cloud/using/api-keys.html. Note that you should
# be using the '--resource cloud' flag.
#
# After generating an API key/secret, to enable Telemetry uncomment the lines below and paste
# in your API key/secret.
#
#confluent.telemetry.enabled=true
#confluent.telemetry.api.key=<CLOUD_API_KEY>
#confluent.telemetry.api.secret=<CCLOUD_API_SECRET>
#### Kafka broker as server to other broker and client
zookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty
zookeeper.ssl.client.enable=true
zookeeper.ssl.protocol=TLSv1.2
zookeeper.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
zookeeper.ssl.truststore.password=confluent
zookeeper.ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
zookeeper.ssl.keystore.password=confluent
ssl.key.password=confluent


#SSL
security.inter.broker.protocol=SSL
ssl.client.enable=required
ssl.protocol=TLSv1.2
ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.truststore.password=confluent
ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
ssl.keystore.password=confluent
ssl.key.password=confluent
root@learn-kafka-2:~#
```

```bash
sudo chown -R cp-kafka:confluent /data1/kafka
```

```bash
sudo chmod -R 755 /data1/kafka
```

```bash
sudo systemctl start confluent-server.service
```
