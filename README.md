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
  <h3>STEP 1: Using docker-compose to bring up zookeeper,kafka,ksqldb-server,ksqldb-cli</h3><hr>
<br>
- Please use the provided docker-compose.yaml file to bring up the complete setup.<br>
- If you already have an existing Zookeeper/Kafka setup, you can remove them from the docker-compose.yaml file and provide the Kafka broker URL inside ksqldb-server's environment configuration (change <code>"KSQL_BOOTSTRAP_SERVERS: kafka:9092"</code> to point to your kafka broker)
<br>
(If you dont have docker-compose already installed, please refer to : https://docs.docker.com/compose/install/)<br>
(Please run <code>docker ps</code> command to check all containers are up and running)

<br><br>
<h3>STEP 2: Create ksqldb Stream and Materialized Views</h3><hr>
<br>
Pre-requisite: Please create the "order" kafka topic using the following command:<br>
<div><pre>docker exec -it kafka /opt/bitnami/kafka/bin/kafka-topics.sh --create --topic order --bootstrap-server localhost:9092</pre></div><br>
1. Connect to ksqldb cli (if all containers are up), to open kqsldb-cli run the following:<br>
  <div><pre>docker exec -it ksqldb-cli ksql http://ksqldb-server:8088</pre></div>
2. Create ksqldb stream:<br>
  <div><pre>CREATE STREAM orderstr (orderId VARCHAR, orderType VARCHAR, orderLines INT)
    WITH (kafka_topic='order', value_format='json', partitions=1);</pre></div>
3. Create Materialized View:<br>
  <div><pre>
  CREATE TABLE myorder AS
  SELECT orderId,
         LATEST_BY_OFFSET(orderType) AS orderType,
         LATEST_BY_OFFSET(orderLines) AS orderLines
  FROM orderstr
  GROUP BY orderId
  EMIT CHANGES;</pre></div>
  
<br><br>
<h3>STEP 3: Inserting data through ksqldb Stream and viewing through select query on Materialized Views</h3><hr>
<br>  
Please use the following data to insert multiple records using ksldb-cli command prompt:<br>
<div><pre>
INSERT INTO orderstr (orderId, orderType, orderLines) VALUES ('ORDER1', 'SHIP', 29);
INSERT INTO orderstr (orderId, orderType, orderLines) VALUES ('ORDER2', 'PICK', 100);
INSERT INTO orderstr (orderId, orderType, orderLines) VALUES ('ORDER3', 'SHIP', 40);
INSERT INTO orderstr (orderId, orderType, orderLines) VALUES ('ORDER4', 'SHIP', 120);
INSERT INTO orderstr (orderId, orderType, orderLines) VALUES ('ORDER5', 'PICK', 50);
INSERT INTO orderstr (orderId, orderType, orderLines) VALUES ('ORDER6', 'SHIP', 90);
INSERT INTO orderstr (orderId, orderType, orderLines) VALUES ('ORDER7', 'PICK', 10);
</pre></div>
<br>
Query the data using a simple select query on ksqldb:
<div><pre>ksql> select * from myOrder where orderType='SHIP';
+---------+-----------+------------+
|ORDERID  |ORDERTYPE  |ORDERLINES  |
+---------+-----------+------------+
|ORDER1   |SHIP       |29          |
|ORDER3   |SHIP       |40          |
|ORDER4   |SHIP       |120         |
|ORDER6   |SHIP       |90          |
</pre></div>
<br>
Query the data using kafka console consumer:
<div><pre>docker exec -it kafka /opt/bitnami/kafka/bin/kafka-console-consumer.sh --topic order --bootstrap-server localhost:9092 --from-beginning
{"ORDERID":"ORDER1","ORDERTYPE":"SHIP","ORDERLINES":29}
{"ORDERID":"ORDER2","ORDERTYPE":"PICK","ORDERLINES":100}
{"ORDERID":"ORDER3","ORDERTYPE":"SHIP","ORDERLINES":40}
{"ORDERID":"ORDER4","ORDERTYPE":"SHIP","ORDERLINES":120}
{"ORDERID":"ORDER5","ORDERTYPE":"PICK","ORDERLINES":50}
{"ORDERID":"ORDER6","ORDERTYPE":"SHIP","ORDERLINES":90}
{"ORDERID":"ORDER7","ORDERTYPE":"PICK","ORDERLINES":10}
</pre></div>

<br><br>
<h3>STEP 4: Inserting data through kafka console producer and viewing through select query on Materialized Views</h3><hr>
<br>

Run the following command, and when you get a <code>></code> prompt, please enter the provided data:
<div><pre>
docker exec -it kafka  /opt/bitnami/kafka/bin/kafka-console-producer.sh --topic order --bootstrap-server localhost:9092
>{"ORDERID":"ORDER8","ORDERTYPE":"SHIP","ORDERLINES":55}
</pre></div><br>
Query the data using a simple select query on ksqldb:<br>
<div><pre>
ksql> select * from myOrder where orderType='SHIP';
+---------+-----------+------------+
|ORDERID  |ORDERTYPE  |ORDERLINES  |
+---------+-----------+------------+
|ORDER1   |SHIP       |29          |
|ORDER3   |SHIP       |40          |
|ORDER4   |SHIP       |120         |
|ORDER6   |SHIP       |90          |
|ORDER8   |SHIP       |55          |
</pre></div>

<br>
Query the data using kafka console consumer:
<div><pre>
docker exec -it kafka /opt/bitnami/kafka/bin/kafka-console-consumer.sh --topic order --bootstrap-server localhost:9092 --from-beginning
{"ORDERID":"ORDER1","ORDERTYPE":"SHIP","ORDERLINES":29}
{"ORDERID":"ORDER2","ORDERTYPE":"PICK","ORDERLINES":100}
{"ORDERID":"ORDER3","ORDERTYPE":"SHIP","ORDERLINES":40}
{"ORDERID":"ORDER4","ORDERTYPE":"SHIP","ORDERLINES":120}
{"ORDERID":"ORDER5","ORDERTYPE":"PICK","ORDERLINES":50}
{"ORDERID":"ORDER6","ORDERTYPE":"SHIP","ORDERLINES":90}
{"ORDERID":"ORDER7","ORDERTYPE":"PICK","ORDERLINES":10}
{"ORDERID":"ORDER8","ORDERTYPE":"SHIP","ORDERLINES":55}
</pre></div>
<br>
So you can clearly see above how we can use ksqldb to both insert and select records into Kafka topic like we are doing it to any relational database.
</P><hr>

<br><br><br>
<h3>References:</h3><br><hr>
Kafka Setup:<br>
https://hub.docker.com/r/bitnami/kafka/
<br><br>
KSQLDB Setup:<br>
https://ksqldb.io/quickstart.html
<br><br>
Unofficial KSQL Python Client:<br>
https://github.com/bryanyang0528/ksql-python
