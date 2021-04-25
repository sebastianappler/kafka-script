# Kafka script

## Usage

Lets assume you are running a kafka cluster in `/kafka` and have an
empty folder `/kafka/ssl` and this repo checked out out in 
`/kafka/script`.

We will generate the private key and a truststore that can be shared
between all brokers and clients in kafka.

Edit the settings for `kafka-generate-key-truststore.sh` and run:
``` sh
cd /kafka/ssl

cp ../script/kafka-generate-key-truststore.sh .
bash kafka-generate-key-truststore.sh

> ls
truststore

>ls trustore
ca-key kafka.truststore.jks
```


Now we will generate and sign a keystore for server0.
If you changed settings in previous script make sure they match the 
settings in this script, for example `CN` must be same.
``` sh
mkdir server0

cp ../script/kafka-generate-keystore.sh server0
cd server0

bash kafka-generate-keystore.sh

>ls
keystore

>ls keystore
kafka.keystore.jks
```

## Config broker and client 

To activate two-way SSL authentication for a server (we call it server0) 
and a client (can be both consumer and producer) we need to add some
settings to the properties-files.

Please note that these aren't complete configurations but these settings the need
to be set in order for SSL to work.

`config/server0.properties`
``` java-properties
listeners=PLAINTEXT://localhost:9092,SSL://localhost:29092

advertised.listeners=PLAINTEXT://localhost:9092,SSL://localhost:29092

security.inter.broker.protocol=SSL
ssl.enabled.protocols=TLSv1.2
ssl.client.auth=required
ssl.keystore.type=JKS
ssl.truststore.type=JKS

ssl.keystore.location=/kafka/ssl/server0/keystore/kafka.keystore.jks
ssl.keystore.password=kafka123
ssl.key.password=kafka123
ssl.truststore.location=/kafka/ssl/truststore/kafka.truststore.jks
ssl.truststore.password=kafka123
```
`config/consumer.properties`
`config/producer.properties`
``` java-properties
bootstrap.servers=localhost:29092

security.protocol=SSL
ssl.keystore.location=/kafka/ssl/client/keystore/kafka.keystore.jks
ssl.keystore.password=kafka123
ssl.key.password=kafka123
ssl.truststore.location=/kafka/ssl/truststore/kafka.truststore.jks
ssl.truststore.password=kafka123
```

