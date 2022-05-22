# kafka-ksqldb
Query Kafka with SQL
<br>
<P>
What is ksqlDB?<br>
ksqlDB is built on top of Kafka Streams, a lightweight, powerful Java library for enriching, transforming, and processing real-time streams of data. Having Kafka Streams at its core means ksqlDB is built on well-designed and easily understood layers of abstractions.
<br><br>
In this tutorial, we are trying to:<br>
  &nbsp;&nbsp;1) setup ksqlDB (most of the information, like docker-compose.yaml can also be taken from their website mentioned below)<br>
  &nbsp;&nbsp;2) Insert data into a Kafka topic from <br> &nbsp;&nbsp;&nbsp;&nbsp;a) ksqldb <br> &nbsp;&nbsp;&nbsp;&nbsp;b) kafka console producer.<br>
  &nbsp;&nbsp;3) Query Data from ksqldb-cli (this can also be done with ksql java/python clients)
<br><br>
Nothing too revolutionary being done here, but this gives an idea about how simple it is to use ksqldb to query a pre-existing kafka broker for data. Its pretty difficult in real life, unless you run console consumer (or any other tool like kafkacat) and read all messages and then filter the data.
<br><br>
STEP 1: Using docker-compose to bring up zookeeper, kafka, ksqldb-server, ksqldb-cli<hr>
<br>
- Please use the provided docker-compose.yaml file to bring up the complete setup.<br>
- If you already have an existing Zookeeper/Kafka setup, you can remove them from the docker-compose.yaml file and provide the Kafka broker URL inside ksqldb-server's environment configuration (change "KSQL_BOOTSTRAP_SERVERS: kafka:9092" to point to your kafka broker)
<br>
(If you dont have docker-compose already installed: https://docs.docker.com/compose/install/)
  
</P>

<br><br><br>
Kafka Setup:<br>
https://hub.docker.com/r/bitnami/kafka/
<br><br>
KSQLDB Setup:<br>
https://ksqldb.io/quickstart.html
<br><br>
Unofficial KSQL Python Client:<br>
https://github.com/bryanyang0528/ksql-python
