# Memasang Security dengan SCRAM-SHA-256
## Kafka REST
Buat file kafka-rest.properties di path /etc/kafka
```bash
#
# Copyright 2018 Confluent Inc.
#
# Licensed under the Confluent Community License (the "License"); you may not use
# this file except in compliance with the License.  You may obtain a copy of the
# License at
#
# http://www.confluent.io/confluent-community-license
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OF ANY KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations under the License.
#

#id=kafka-rest-test-server
#schema.registry.url=http://localhost:8081
#zookeeper.connect=localhost:2181
bootstrap.servers=SASL_SSL://node1.kafka:9092
#
# Configure interceptor classes for sending consumer and producer metrics to Confluent Control Center
# Make sure that monitoring-interceptors-<version>.jar is on the Java class path
#consumer.interceptor.classes=io.confluent.monitoring.clients.interceptor.MonitoringConsumerInterceptor
#producer.interceptor.classes=io.confluent.monitoring.clients.interceptor.MonitoringProducerInterceptor


#SSL config
# security.protocol=SSL
ssl.protocol=TLSv1.2
ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.truststore.password=confluent
ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
ssl.keystore.password=confluent
ssl.key.password=confluent
security.protocol=SASL_SSL
ssl.keystore.type=PKCS12
ssl.truststore.type=PKCS12

#SASL config

sasl.mechanism=SCRAM-SHA-256

# JAAS config for SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="cp-kafka" \
    password="confluentdua";
```
# Ksql

```bash
#
# Copyright 2018 Confluent Inc.
#
# Licensed under the Confluent Community License (the "License"); you may not use
# this file except in compliance with the License.  You may obtain a copy of the
# License at
#
# http://www.confluent.io/confluent-community-license
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OF ANY KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations under the License.
#

#------ Endpoint config -------

### HTTP  ###
# The URL the KSQL server will listen on:
# The default is any IPv4 interface on the machine.
# NOTE: If set to wildcard or loopback set 'advertised.listener' to enable pull queries across machines
listeners=http://node1.kafka:8088

# Use the 'listeners' line below for any IPv6 interface on the machine.
# listeners=http://[::]:8088

# If running a multi-node cluster across multiple machines and 'listeners' is set to a wildcard or loopback address
# 'advertised.listener' must be set to the URL other KSQL nodes should use to reach this node.
# advertised.listener=?

### HTTPS ###
# To switch KSQL over to communicating using HTTPS comment out the 'listeners' line above
# uncomment and complete the properties below.
# See: https://docs.ksqldb.io/en/latest/operate-and-deploy/installation/server-config/security/#configure-the-cli-for-https
#
# listeners=https://0.0.0.0:8088
# advertised.listener=?
# ssl.truststore.location=?
# ssl.truststore.password=?
# ssl.keystore.location=?
# ssl.keystore.password=?
# ssl.key.password=?

#------ Logging config -------

# Automatically create the processing log topic if it does not already exist:
ksql.logging.processing.topic.auto.create=true

# Automatically create a stream within KSQL for the processing log:
ksql.logging.processing.stream.auto.create=true

# Uncomment the following if you wish the errors written to the processing log to include the
# contents of the row that caused the error.
# Note: care should be taken to restrict access to the processing topic if the data KSQL is
# processing contains sensitive information.
#ksql.logging.processing.rows.include=true

#------- Metrics config --------

# Turn on collection of metrics of function invocations:
# ksql.udf.collect.metrics=true

#------ External service config -------

#------ Kafka -------

# The set of Kafka brokers to bootstrap Kafka cluster information from:
bootstrap.servers=node1.kafka:9092,node2.kafka:9092,node3.kafka:9092 

ssl.protocol=TLSv1.2
ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.truststore.password=confluent
# ssl.keystore.location=path/to/kafka-server/producer/client.ks.p12
# ssl.keystore.password=yourpassword
# ssl.key.password=yourpassword
security.protocol=SASL_SSL
ssl.keystore.type=PKCS12

sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="cp-kafka" password="confluentdua";

# Enable snappy compression for the Kafka producers
compression.type=snappy

#------ Connect -------

# uncomment the below to start an embedded Connect worker
# ksql.connect.worker.config=config/connect.properties

#------ Schema Registry -------

# Uncomment and complete the following to enable KSQL's integration to the Confluent Schema Registry:
# ksql.schema.registry.url=http://localhost:8081
state.dir=/dataku/ksql/



# Enable KSQL to report metrics to Control Center
confluent.controlcenter.metrics.topic=__confluent.metrics
confluent.controlcenter.metrics.bootstrap.servers=node1.kafka:9092,node2.kafka:9092,node3.kafka:9092
```

## Schema Registry

```bash
#
# Copyright 2018 Confluent Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# The address the socket server listens on.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=http://node1.kafka:8081

# The host name advertised in ZooKeeper. This must be specified if your running Schema Registry
# with multiple nodes.
# host.name=your-ip-hostname
# Use this setting to specify the bootstrap servers for your Kafka cluster and it
# will be used both for selecting the leader schema registry instance and for storing the data for
# registered schemas.
# kafkastore.bootstrap.servers=SASL_SSL://your-ip-hostname:9092,SASL_SSL://your-ip-hostname:9092,SASL_SSL://your-ip-hostname:9092,
kafkastore.bootstrap.servers=SASL_SSL://node1.kafka:9092,SASL_SSL://node2.kafka:9092,SASL_SSL://node3.kafka:9092
# The name of the topic to store schemas in
kafkastore.topic=_schemas

# If true, API requests that fail will include extra debugging information, including stack traces
debug=false

metadata.encoder.secret=REPLACE_ME_WITH_HIGH_ENTROPY_STRING

resource.extension.class=io.confluent.dekregistry.DekRegistryResourceExtension

kafkastore.security.protocol=SASL_SSL
kafkastore.ssl.protocols=TLSv1.2
kafkastore.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
kafkastore.ssl.truststore.password=confluent
# kafkastore.ssl.keystore.location=path/to/kafka-server/schema-registry/client.ks.p12
# kafkastore.ssl.keystore.password=yourpassword
# ssl.keystore.type=PKCS12
# ssl.endpoint.identification.algorithm=
#
kafkastore.sasl.mechanism=SCRAM-SHA-256
kafkastore.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="cp-kafka" password="confluentdua";
```

## Kafka Connect
Pengaturan untuk connect distributed
```bash
##
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
##

# This file contains some of the configurations for the Kafka Connect distributed worker. This file is intended
# to be used with the examples, and some settings may differ from those used in a production system, especially
# the `bootstrap.servers` and those specifying replication factors.

# A list of host/port pairs to use for establishing the initial connection to the Kafka cluster.
bootstrap.servers=SASL_SSL://node1.kafka:9092

# unique name for the cluster, used in forming the Connect cluster group. Note that this must not conflict with consumer group IDs
group.id=connect-cluster

# The converters specify the format of data in Kafka and how to translate it into Connect data. Every Connect user will
# need to configure these based on the format they want their data in when loaded from or stored into Kafka
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
# Converter-specific settings can be passed in by prefixing the Converter's setting with the converter we want to apply
# it to
key.converter.schemas.enable=true
value.converter.schemas.enable=true

# Topic to use for storing offsets. This topic should have many partitions and be replicated and compacted.
# Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
offset.storage.topic=connect-offsets
offset.storage.replication.factor=1
#offset.storage.partitions=25

# List of comma-separated URIs the REST API will listen on.
listeners=http://0.0.0.0:8083 
rest.advertised.host.name=node1.kafka
rest.advertised.port=8083

# Topic to use for storing connector and task configurations; note that this should be a single partition, highly replicated,
# and compacted topic. Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
config.storage.topic=connect-configs
config.storage.replication.factor=1

# Topic to use for storing statuses. This topic can have multiple partitions and should be replicated and compacted.
# Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
status.storage.topic=connect-status
status.storage.replication.factor=1
#status.storage.partitions=5

# Flush much faster than normal, which is useful for testing/debugging
offset.flush.interval.ms=10000

# List of comma-separated URIs the REST API will listen on. The supported protocols are HTTP and HTTPS.
# Specify hostname as 0.0.0.0 to bind to all interfaces.
# Leave hostname empty to bind to default interface.
# Examples of legal listener lists: HTTP://myhost:8083,HTTPS://myhost:8084"
#listeners=HTTP://:8083

# The Hostname & Port that will be given out to other workers to connect to i.e. URLs that are routable from other servers.
# If not set, it uses the value for "listeners" if configured.
#rest.advertised.host.name=
#rest.advertised.port=
#rest.advertised.listener=

# Set to a list of filesystem paths separated by commas (,) to enable class loading isolation for plugins
# (connectors, converters, transformations). The list should consist of top level directories that include 
# any combination of: 
# a) directories immediately containing jars with plugins and their dependencies
# b) uber-jars with plugins and their dependencies
# c) directories immediately containing the package directory structure of classes of plugins and their dependencies
# Examples: 
# plugin.path=/usr/local/share/java,/usr/local/share/kafka/plugins,/opt/connectors,
#plugin.path=/usr/share/java,/usr/share/confluent-hub-components
plugin.path=/usr/share/java,/usr/share/confluent-hub-components

#SSL config
security.protocol=SASL_SSL
ssl.protocol=TLSv1.2
ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.truststore.password=confluent
ssl.truststore.type=PKCS12
ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
ssl.keystore.password=confluent
ssl.keystore.type=PKCS12

#SASL config

sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=\"cp-kafka\" password=\"confluentdua\";

#Property for Authorization

authorizer.class.name=kafka.security.authorizer.AclAuthorizer


# Authentication settings for Connect producers used with source connectors
producer.security.protocol=SASL_SSL
producer.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
producer.ssl.truststore.password=confluent
producer.ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
producer.ssl.keystore.password=confluent
producer.sasl.mechanism=SCRAM-SHA-256
producer.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=\"alice\" password=\"alicetiga\";

producer.confluent.monitoring.interceptor.sasl.mechanism=SCRAM-SHA-256
producer.confluent.monitoring.interceptor.security.protocol=SASL_SSL

# Authentication settings for Connect consumers used with sink connectors
consumer.security.protocol=SASL_SSL
consumer.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
consumer.ssl.truststore.password=confluent
consumer.ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
consumer.ssl.keystore.password=confluent
consumer.sasl.mechanism=SCRAM-SHA-256
consumer.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=\"bob\" password=\"bobtiga\";

consumer.confluent.monitoring.interceptor.sasl.mechanism=SCRAM-SHA-256
consumer.confluent.monitoring.interceptor.security.protocol=SASL_SSL
```

## Confluent Control Center

```bash
# (Copyright) Confluent, Inc.

# These configs are designed to make control center's system requirements as low as 
# reasonably possible. It is still capable of monitoring a moderate number of resources,
# but it specifically trades off throughput in favor of low CPU load.

############################# Server Basics #############################

# A comma separated list of Apache Kafka cluster host names (required)
bootstrap.servers=node1.kafka:9092,node2.kafka:9092,node3.kafka:9092

# A comma separated list of ZooKeeper host names (for ACLs)
zookeeper.connect=node1.zookeeper:2182,node2.zookeeper:2182,node3.zookeeper:2182

############################# Control Center Settings #############################

# Unique identifier for the Control Center
confluent.controlcenter.id=1

# Directory for Control Center to store data
confluent.controlcenter.data.dir=/dataku/control-center

# License string for the Control Center
# confluent.license=Xyz

confluent.controlcenter.rest.listeners=http://10.100.13.208:9021
#listeners=http://node1.kafka:9021


# A comma separated list of Connect host names
confluent.controlcenter.connect.connect-cluster.cluster=http://node1.kafka:8083

# KSQL cluster URL
confluent.controlcenter.ksql.connect-cluster.url=http://node1.kafka:8088

# Schema Registry cluster URL
confluent.controlcenter.schema.registry.url=http://node1.kafka:8081

# Kafka REST endpoint URL
confluent.controlcenter.streams.cprest.url=http://node1.kafka:8090

# Settings to enable email alerts
#confluent.controlcenter.mail.enabled=true
#confluent.controlcenter.mail.host.name=smtp1
#confluent.controlcenter.mail.port=587
#confluent.controlcenter.mail.from=kafka-monitor@example.com
#confluent.controlcenter.mail.password=abcdefg
#confluent.controlcenter.mail.starttls.required=true

# Replication for internal Control Center topics.
# Only lower them for testing.
# WARNING: replication factor of 1 risks data loss.
confluent.controlcenter.internal.topics.replication=1

# Number of partitions for Control Center internal topics
# Increase for better throughput on monitored data (CPU bound)
# NOTE: changing requires running `bin/control-center-reset` prior to restart
confluent.controlcenter.internal.topics.partitions=1

# Topic used to store Control Center configuration
# WARNING: replication factor of 1 risks data loss.
confluent.controlcenter.command.topic.replication=1

# Enable automatic UI updates
confluent.controlcenter.ui.autoupdate.enable=true

# Enable usage data collection
confluent.controlcenter.usage.data.collection.enable=true

# Enable Controller Chart in Broker page
#confluent.controlcenter.ui.controller.chart.enable=true

############################# Control Center RBAC Settings #############################

# Enable RBAC authorization in Control Center by providing a comma-separated list of Metadata Service (MDS) URLs
#confluent.metadata.bootstrap.server.urls=http://localhost:8090

# MDS credentials of an RBAC user for Control Center to act on behalf of
# NOTE: This user must be a SystemAdmin on each Apache Kafka cluster
#confluent.metadata.basic.auth.user.info=username:password

# Enable SASL-based authentication for each Apache Kafka cluster (SASL_PLAINTEXT or SASL_SSL required)
#confluent.controlcenter.streams.security.protocol=SASL_PLAINTEXT
#confluent.controlcenter.kafka.<name>.security.protocol=SASL_PLAINTEXT

# Enable authentication using a bearer token for Control Center's REST endpoints
#confluent.controlcenter.rest.authentication.method=BEARER

# Public key used to verify bearer tokens
# NOTE: Must match the MDS public key
#public.key.path=/path/to/publickey.pem

############################# Broker (Metrics reporter) Monitoring #############################

# Set how far back in time metrics reporter data should be processed
#confluent.metrics.topic.skip.backlog.minutes=15

############################# Stream (Interceptor) Monitoring #############################

# Keep these settings default unless using non-Confluent interceptors

# Override topic name for intercepted (should mach custom interceptor settings)
#confluent.monitoring.interceptor.topic=_confluent-monitoring

# Number of partitions for the intercepted topic
confluent.monitoring.interceptor.topic.partitions=1

# Amount of replication for intercepted topics
# WARNING: replication factor of 1 risks data loss.
confluent.monitoring.interceptor.topic.replication=1

# Set how far back in time interceptor data should be processed
#confluent.monitoring.interceptor.topic.skip.backlog.minutes=15

############################# System Health (Broker) Monitoring #############################

# Number of partitions for the metrics topic
confluent.metrics.topic.partitions=1

# Replication factor for broker monitoring data
# WARNING: replication factor of 1 risks data loss.
confluent.metrics.topic.replication=1

############################# Streams (state store) settings #############################

# Increase for better throughput on data processing (CPU bound)
confluent.controlcenter.streams.num.stream.threads=1

# Amount of heap to use for internal caches. Increase for better thoughput
confluent.controlcenter.streams.cache.max.bytes.buffering=100000000

#Confluent streams security settings
confluent.controlcenter.streams.security.protocol=SASL_SSL
# confluent.controlcenter.streams.ssl.keystore.location=path/to/kafka-server/control-center/client.ks.p12
# confluent.controlcenter.streams.ssl.keystore.password=yourpassword
confluent.controlcenter.streams.ssl.key.password=confluent
confluent.controlcenter.streams.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
confluent.controlcenter.streams.ssl.truststore.password=confluent
confluent.controlcenter.streams.sasl.mechanism=SCRAM-SHA-256
confluent.controlcenter.streams.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="cp-kafka" password="confluentdua";


#Confluent schema security settings
confluent.controlcenter.schema.registry.schema.registry.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
confluent.controlcenter.schema.registry.schema.registry.ssl.truststore.password=confluent
confluent.controlcenter.schema.registry.schema.registry.ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
confluent.controlcenter.schema.registry.schema.registry.ssl.keystore.password=confluent
confluent.controlcenter.schema.registry.schema.registry.ssl.key.password=confluent



#Confluent rest security settings
#confluent.controlcenter.rest.security.protocol=SASL_SSL
confluent.controlcenter.rest.ssl.protocol=TLSv1.2
confluent.controlcenter.rest.proxy.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
confluent.controlcenter.rest.proxy.ssl.truststore.password=confluent
confluent.controlcenter.rest.proxy.ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
confluent.controlcenter.rest.proxy.ssl.keystore.password=confluent
confluent.controlcenter.rest.proxy.ssl.key.password=confluent
#


#Confluent kafka-connect security settings
confluent.controlcenter.connect.connect.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
confluent.controlcenter.connect.connect.ssl.truststore.password=confluent




#Confluent ksqldb security settings
confluent.controlcenter.ksql.ksql.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
confluent.controlcenter.ksql.ksql.ssl.truststore.password=confluent
```

## Server Properties
Sesuaikan dengan 3 node, seperti listenernya
```bash
############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

#   FORMAT:
#  listeners = listener_name://host_name:port
listeners=SASL_SSL://:9092
#listeners=SSL://:9092
#listeners=PLAINTEXT://:9092

# Listener name, hostname and port the broker will advertise to clients.
advertised.listeners=SASL_SSL://node1.kafka:9092
#advertised.listeners=SSL://node1.kafka:9092

#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
listener.security.protocol.map=SASL_SSL:SASL_SSL
#listener.security.protocol.map=SSL:SSL

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
log.dirs=/dataku/kafka

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


# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

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
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interva     l.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, re     balances during application startup.
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
ssl.client.enable=none
ssl.protocol=TLSv1.2
ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.truststore.password=confluent
ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
ssl.keystore.password=confluent
ssl.key.password=confluent


#SASL

# SASL Configuration
#security.inter.broker.protocol=SSL

sasl.enabled.mechanisms=SCRAM-SHA-256
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
security.inter.broker.protocol=SASL_SSL
listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="cp-kafka" password="confluentdua";
security.protocol=SASL_SSL
super.users=User:cp-kafka
#Property for Authorization

authorizer.class.name=kafka.security.authorizer.AclAuthorizer

allow.everyone.if.no.acl.found=true

# Use for metrics

##################### Confluent Metrics Reporter #######################
# Confluent Control Center and Confluent Auto Data Balancer integration
#
# Uncomment the following lines to publish monitoring data for
# Confluent Control Center and Confluent Auto Data Balancer
# If you are using a dedicated metrics cluster, also adjust the settings
# to point to your metrics kakfa cluster.
metric.reporters=io.confluent.metrics.reporter.ConfluentMetricsReporter
confluent.metrics.reporter.bootstrap.servers=node1.kafka:9092,node2.kafka:9092,node3.kafka:9092
#
# Uncomment the following line if the metrics cluster has a single broker
confluent.metrics.reporter.topic.replicas=3
confluent.metrics.reporter.security.protocol=SASL_SSL
confluent.metrics.reporter.ssl.keystore.location=/var/ssl/private/kafka.server.keystore.jks
confluent.metrics.reporter.ssl.keystore.password=confluent
confluent.metrics.reporter.ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
# confluent.metrics.reporter.ssl.endpoint.identification.algorithm=
# confluent.metrics.reporter.ssl.client.auth=required
confluent.metrics.reporter.ssl.truststore.password=confluent
confluent.metrics.reporter.ssl.key.password=confluent

confluent.metrics.reporter.sasl.mechanism=SCRAM-SHA-256
confluent.metrics.reporter.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="cp-kafka" password="confluentdua";


# Mengaktifkan JMX
#JMX_PORT=1099
```
