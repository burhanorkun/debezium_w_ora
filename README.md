## Debezium From Oracle To Oracle

You must download the [Oracle instant client for Linux](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)
and put it under the directory _debezium-with-oracle-jdbc/oracle_instantclient_.

```shell
                   +-------------+
                   |             |
                   |   Oracle    |
                   |             |
                   +-------------+
                          +
                          |
                          |
                          |
                          v
          +----------------------------------+
          |                                  |
          |           Kafka Connect          |
          |  (Debezium, JDBC connectors)     |
          |                                  |
          +----------------------------------+
                          +
                          |
                          |
                          |
                          v
                 +-----------------+
                 |                 |
                 |  TARGET ORACLE  |
                 |                 |
                 +-----------------+
```

- Start the topology as defined in <https://debezium.io/docs/tutorial/>

```shell
export DEBEZIUM_VERSION=1.6
docker-compose -f docker-compose.yaml up --build
```

- Start Oracle sink connector for Customers table.

```shell
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-oracle-sink-customers.json
```

- Start Oracle source connector

```shell
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-source-oracle.json
```

- Connect to Source Oracle DB
    - Host: localhost
    - Port: 1521
    - Service Name: XE
    - user: SYS
    - pass: oracle

- Connect to Target Oracle DB
    - Host: localhost
    - Port: 3042
    - Service Name: XE
    - user: SYS
    - pass: oracle

- Make changes on Source DB, see results on kafka topic, and on the target database.

- See the kafka topics

```shell
docker exec -it kafka /kafka/bin/kafka-topics.sh --bootstrap-server kafka:9092 --list
```

- Inpsect a kafka topic

```shell
export DEBEZIUM_VERSION=1.6
docker-compose -f docker-compose.yaml exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbz_oracle.INVENTORY.CUSTOMERS
```

- See the connectors

```shell
curl -i -X GET  http://localhost:8083/connectors
```

- Manage Connectors
  - See the connector status

    ```shell
    curl -s "http://localhost:8083/connectors?expand=info&expand=status"
    ```

  - Restart a connector

    ```shell
    curl -i -X POST  http://localhost:8083/connectors/inventory-source-connector/restart
    #OR 
    curl -i -X POST  http://localhost:8083/connectors/jdbc-sink-customers/restart
    ```

  - Remove a connector

    ```shell
    curl -i -X DELETE  http://localhost:8083/connectors/inventory-source-connector
    #OR 
    curl -i -X DELETE  http://localhost:8083/connectors/jdbc-sink-customers
    ```

- Stop the topology

```shell
export DEBEZIUM_VERSION=1.6
docker-compose -f docker-compose.yaml down
```
