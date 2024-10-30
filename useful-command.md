

Services
```bash
systemctl list-units --type=service --all
```

Keamanan


## ACLs
Produce message
```bash
kafka-console-producer --topic ACLS --bootstrap-server node3.kafka:9092 --producer.config produce.properties
```
Consume message
```bash
kafka-console-consumer --topic ACLS --bootstrap-server node3.kafka:9092 --consumer.config consumer.properties
```
