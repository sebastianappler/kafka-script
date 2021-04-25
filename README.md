# Kafka script

## Kafka & clients with SSL
### Prequsites
You need to have `Apache kafka` and `java` available on your system.

Install java:
``` sh
sudo apt install openjdk-11-jdk-headless
```

Manual install of apache kafka:
``` sh
curl -LO https://downloads.apache.org/kafka/2.8.0/kafka_2.13-2.8.0.tgz
tar -xvf kafka_2.13-2.8.0.tgz
mv kafka_2.13-2.8.0 ~/kafka
```

### Usage
We will generate the private key and a truststore that can be shared
between all brokers and clients in kafka.

Edit the settings for `kafka-generate-key-truststore.sh` and run:
``` sh
cd ~/kafka
git clone https://github.com/sebastianappler/kafka-script script

mkdir ssl
cd ssl

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

### Config broker and client 

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

ssl.keystore.location/home/you/kafka/ssl/server0/keystore/kafka.keystore.jks
ssl.keystore.password=kafka123
ssl.key.password=kafka123
ssl.truststore.location/home/you/kafka/ssl/truststore/kafka.truststore.jks
ssl.truststore.password=kafka123
```
`config/consumer.properties` & `config/producer.properties`
``` java-properties
bootstrap.servers=localhost:29092

security.protocol=SSL
ssl.keystore.location/home/you/kafka/ssl/client/keystore/kafka.keystore.jks
ssl.keystore.password=kafka123
ssl.key.password=kafka123
ssl.truststore.location/home/you/kafka/ssl/truststore/kafka.truststore.jks
ssl.truststore.password=kafka123
```

### Run kafka with ssl
To run the apache kafka scripts make sure you add `$HOME/kafka/bin` to your `$PATH`.

``` sh
# Start Zookeeper and Kafka Broker
zookeeper-server-start.sh -daemon ../config/zookeeper.properties
kafka-server-start.sh -daemon ../config/server0.properties

>jps
30161 Kafka
34818 Jps
14220 QuorumPeerMain
# If you dont see any entries when running `jps` you 
# can check `~/kafka/logs/server.log` for errors.

# Topic (if you don't have one already)
kafka-topics.sh --zookeeper localhost:2181 --create --topic test-topic --partitions 1 --replication-factor 1

# Producer and consumer
kafka-console-producer.sh --broker-list localhost:29092 --topic test-topic --producer.config ../config/producer.properties
> Test message!
Ctrl+c

kafka-console-consumer.sh --bootstrap-server localhost:29092 --topic test-topic --from-beginning --consumer.config ../config/consumer.properties
Test message!
```

# Zookeeper & broker SSL

We will generate a different private key since zookeeper has
it's own SSL configuration.

Update the scripts to make the filenames match zookeeper `kafka.truststore.jks`->`zookeeper.truststore.jks`.

``` sh
cd ~/kafka/ssl
mkdir zookeeper
cd zookeeper

cp ~/kafka/script/kafka-generate-key-truststore.sh .
cp ~/kafka/script/kafka-generate-keystore.sh .

# Edit the files and change `kafka.truststore.jks`->`zookeeper.truststore.jks`
# Also feel free to change the password
vi kafka-generate-key-truststore.sh
vi kafka-generate-keystore.sh

bash kafka-generate-key-truststore.sh
bash kafka-generate-keystore.sh

mkdir server0
cd server0

cp ../kafka-generate-keystore.sh .
bash kafka-generate-keystore.sh
```

Update `zookeeper.properties`

``` java-properties
secureClientPort=2182

serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
authProvider.x509=org.apache.zookeeper.server.auth.X509AuthenticationProvider

ssl.keyStore.location=/home/you/kafka/ssl/zookeeper/keystore/zookeeper.keystore.jks
ssl.keyStore.password=zookeeper123
ssl.trustStore.location=/home/you/kafka/ssl/zookeeper/truststore/zookeeper.truststore.jks
ssl.trustStore.password=zookeeper123
```

Update `server0.properties`

``` java-properties

zookeeper.connect=localhost:2182

zookeeper.ssl.client.enable=true
zookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty

zookeeper.ssl.keystore.location=/home/you/kafka/ssl/zookeeper/server0/keystore/zookeeper.keystore.jks
zookeeper.ssl.keystore.password=zookeeper123
zookeeper.ssl.key.password=zookeeper123
zookeeper.ssl.truststore.location=/home/you/kafka/ssl/zookeeper/truststore/zookeeper.truststore.jks
zookeeper.ssl.truststore.password=zookeeper123

zookeeper.set.acl=true
```

Now you should be able to run the examples from [Run kafka with ssl](###-run-kafka-with-ssl).
Note that the zookeeper port 2181 can still be used for `kafka-topic.sh`.
