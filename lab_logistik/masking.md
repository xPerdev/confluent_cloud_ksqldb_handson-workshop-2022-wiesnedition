# Retail and Logistic case: Masking PII Data (steps for Confluent Cloud)


We are going to build data pipeline which should look like this:
![Retail Use cases as flow](img/Mask1.png)

## 1. First Steps
- Login to Confluent Cloud.
- Select your environment and your Cluster.
- From the left panel select "ksqlDB" to display all apps.
- Select your ksqlDB cluster to display the ksqlDB Editor.

  ![Start Screen](img/Mask2.png)



## 2. Create Streams 

Suppose you have a topic that contains personally identifiable information (PII), and you want to mask those fields. In this tutorial, we'll write a program that persists the events in the original topic to a new Kafka topic with the PII removed or obfuscated.

First, you’ll need to create a Kafka topic and stream to represent the purchases data. The following creates both in one shot.

Enter following commands (click button "Run query" for each command):

```
CREATE STREAM purchases (order_id INT, customer_name VARCHAR, date_of_birth VARCHAR,
                         product VARCHAR, order_total_usd DOUBLE, town VARCHAR, country VARCHAR)
    WITH (kafka_topic='purchases', value_format='json', partitions=1);
```


## 3. Insert Data to Streams

In the ksqlDB Editor add some data to your streams.

purchase data using the following commands:
```
INSERT INTO purchases (order_id, customer_name, date_of_birth, product, order_total_usd, town, country) VALUES (1, 'Britney', '02/29/2000', 'Heart Rate Monitor', 119.93, 'Denver', 'USA');
INSERT INTO purchases (order_id, customer_name, date_of_birth, product, order_total_usd, town, country) VALUES (2, 'Michael', '06/08/1981', 'Foam Roller', 34.95, 'Los Angeles', 'USA');
INSERT INTO purchases (order_id, customer_name, date_of_birth, product, order_total_usd, town, country) VALUES (3, 'Kimmy', '05/19/1978', 'Hydration Belt', 50.00, 'Tuscan', 'USA');
INSERT INTO purchases (order_id, customer_name, date_of_birth, product, order_total_usd, town, country) VALUES (4, 'Samantha', '08/05/1983', 'Wireless Headphones', 175.93, 'Tulsa', 'USA');
INSERT INTO purchases (order_id, customer_name, date_of_birth, product, order_total_usd, town, country) VALUES (5, 'Jonathon', '01/31/1981', 'Comfort Insoles', 49.95, 'Portland', 'USA');
INSERT INTO purchases (order_id, customer_name, date_of_birth, product, order_total_usd, town, country) VALUES (6, 'Raymond', '07/29/2001', 'Running Beanie', 13.73, 'Omaha', 'USA');
```


## 4. Verify the entered data

Please set the following query property to set ksqlDB to query data from the beginning of the topic.
* ```auto.offset.reset``` to 'Earliest'

![Needed Properties](img/Mask3.png)



Now we should be able to see all of the purchases data we just entered with the following command:
- The **SELECT** statement pushes a continuous stream of updates from the ksqlDB stream or table. The result is not persisted in a Kafka topic and is printed out only in the console.


```
SELECT *
    FROM purchases
    EMIT CHANGES
    LIMIT 6;
```
This should yield roughly the following output. The order will be different depending on how the records were actually inserted. Note that PII like name, birthdate, city, and country are present.

```bash
+--------------------+--------------------+--------------------+--------------------+--------------------+--------------------+--------------------+
|ORDER_ID            |CUSTOMER_NAME       |DATE_OF_BIRTH       |PRODUCT             |ORDER_TOTAL_USD     |TOWN                |COUNTRY             |
+--------------------+--------------------+--------------------+--------------------+--------------------+--------------------+--------------------+
|1                   |Britney             |02/29/2000          |Heart Rate Monitor  |119.93              |Denver              |USA                 |
|2                   |Michael             |06/08/1981          |Foam Roller         |34.95               |Los Angeles         |USA                 |
|3                   |Kimmy               |05/19/1978          |Hydration Belt      |50.0                |Tuscan              |USA                 |
|4                   |Samantha            |08/05/1983          |Wireless Headphones |175.93              |Tulsa               |USA                 |
|5                   |Jonathon            |01/31/1981          |Comfort Insoles     |49.95               |Portland            |USA                 |
|6                   |Raymond             |07/29/2001          |Running Beanie      |13.73               |Omaha               |USA                 |
Limit Reached
Query terminated
```

## 5. Handle PII Data

Next we will highlight two ways to mask PII data, both methods will result in new streams.
Our first masking technique will be to create a derived topic in which all PII is excluded. This technique masks data by refraining from pulling in PII fields like CUSTOMER_NAME and DATE_OF_BIRTH.

- creating a new materialized stream view with a corresponding new Kafka sink topic (`purchases_pii_removed`) from `purchases` source topic
- the newly created topic can be used in further queries
```
CREATE STREAM purchases_pii_removed
    WITH (kafka_topic='purchases_pii_removed', value_format='json', partitions=1) AS
    SELECT ORDER_ID, PRODUCT, ORDER_TOTAL_USD, TOWN, COUNTRY
    FROM PURCHASES;
```
Let’s verify that the derived topic we just created does not have any PII related to CUSTOMER_NAME or DATE_OF_BIRTH. You can see the contents of the stream by executing the following:

```
SELECT *
    FROM purchases_pii_removed
    EMIT CHANGES
    LIMIT 6;
```

Your results should look like what is below. Take note of the lack of PII fields like CUSTOMER_NAME or DATE_OF_BIRTH.

```
+--------------------+--------------------+--------------------+--------------------+--------------------+
|ORDER_ID            |PRODUCT             |ORDER_TOTAL_USD     |TOWN                |COUNTRY             |
+--------------------+--------------------+--------------------+--------------------+--------------------+
|1                   |Heart Rate Monitor  |119.93              |Denver              |USA                 |
|2                   |Foam Roller         |34.95               |Los Angeles         |USA                 |
|3                   |Hydration Belt      |50.0                |Tuscan              |USA                 |
|4                   |Wireless Headphones |175.93              |Tulsa               |USA                 |
|5                   |Comfort Insoles     |49.95               |Portland            |USA                 |
|6                   |Running Beanie      |13.73               |Omaha               |USA                 |
Limit Reached
Query terminated
```
The second technique for masking data utilizes ksqlDB’s built in MASK functions. Here we retain the customer name and date of birth, but obfuscated.

```
CREATE STREAM purchases_pii_obfuscated
    WITH (kafka_topic='purchases_pii_obfuscated', value_format='json', partitions=1) AS
    SELECT MASK(CUSTOMER_NAME) AS CUSTOMER_NAME,
           MASK(DATE_OF_BIRTH) AS DATE_OF_BIRTH,
           ORDER_ID, PRODUCT, ORDER_TOTAL_USD, TOWN, COUNTRY
    FROM PURCHASES;
```

Use the command below to query the contents of the purchases_pii_obfuscated stream:

```
SELECT *
    FROM purchases_pii_obfuscated
    EMIT CHANGES
    LIMIT 6;
```

We can see that the sensitive data is masked with x’s or n’s.

```

+--------------------+--------------------+--------------------+--------------------+--------------------+--------------------+--------------------+
|CUSTOMER_NAME       |DATE_OF_BIRTH       |ORDER_ID            |PRODUCT             |ORDER_TOTAL_USD     |TOWN                |COUNTRY             |
+--------------------+--------------------+--------------------+--------------------+--------------------+--------------------+--------------------+
|Xxxxxxx             |nn-nn-nnnn          |1                   |Heart Rate Monitor  |119.93              |Denver              |USA                 |
|Xxxxxxx             |nn-nn-nnnn          |2                   |Foam Roller         |34.95               |Los Angeles         |USA                 |
|Xxxxx               |nn-nn-nnnn          |3                   |Hydration Belt      |50.0                |Tuscan              |USA                 |
|Xxxxxxxx            |nn-nn-nnnn          |4                   |Wireless Headphones |175.93              |Tulsa               |USA                 |
|Xxxxxxxx            |nn-nn-nnnn          |5                   |Comfort Insoles     |49.95               |Portland            |USA                 |
|Xxxxxxx             |nn-nn-nnnn          |6                   |Running Beanie      |13.73               |Omaha               |USA                 |
Limit Reached
Query terminated
```


Now check in Confluent Cloud UI:
* check in ksqlDB Cluster - the persistent queries. Take a look in the details (SINK: and SOURCE:) of the running queries.
* check performance tab if *Query Saturation* and *Disk Usage* graphs are displaying activity.
* check in ksqlDB cluster the flow to follow the expansion easier. If it is not visible refresh the webpage in browser.

**Persistent queries** are server-side queries that run indefinitely processing rows of events. You issue persistent queries by deriving new streams and new tables from existing streams or tables.

![Persistent Queries](img/Mask4.png)

END of Masking Lab.


[Back](../README.md#Agenda) to Agenda.
