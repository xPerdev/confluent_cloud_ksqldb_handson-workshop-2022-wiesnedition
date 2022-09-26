# Internet of Things Use Case: Temperature Alerting System (Confluent Cloud)

We want to build an alerting system that automatically detects if the temperature of a room consistently drops.
We'll write a program that monitors a stream of temperature readings and detects when the temperature
consistently drops below 45 degrees Fahrenheit for a period of 10 minutes.

Our data pipeline should look like this:
![ Temperature Alerting System Flow](img_temperature_alerting_system/datapipeline.png)

## 1 Confluent Cloud ksqldb setup to enable use run our ksqlDB script.

- Login to Confluent Cloud.
- Select or create an environment.
- Create new ksqlDB cluster or select an existing from within your environment.
- Select "ksqlDB" from your left panel to display all ksqlDb clusters.
- Select a ksqlDB cluster to display ksqlDB editor.

![Start Screen](img_temperature_alerting_system/ksqlDB_Start.png)

## 2. Create stream (TEMPERATURE_READINGS), this equally auto creates topic (TEMPERATURE_READINGS)

```
CREATE STREAM TEMPERATURE_READINGS (ID VARCHAR KEY, TIMESTAMP VARCHAR, READING BIGINT)
    WITH (KAFKA_TOPIC = 'TEMPERATURE_READINGS',
          VALUE_FORMAT = 'JSON',
          TIMESTAMP = 'TIMESTAMP',
          TIMESTAMP_FORMAT = 'yyyy-MM-dd HH:mm:ss',
          PARTITIONS = 1);
```

Check created stream and topic from stream and topic tabs within your cloud UI.

## 3. Insert sample data into create topic. Which would be accessed by our stream.

```
INSERT INTO TEMPERATURE_READINGS (ID, TIMESTAMP, READING) VALUES ('1', '2022-09-23 02:15:30', 55);
INSERT INTO TEMPERATURE_READINGS (ID, TIMESTAMP, READING) VALUES ('1', '2022-09-23 02:20:30', 50);
INSERT INTO TEMPERATURE_READINGS (ID, TIMESTAMP, READING) VALUES ('1', '2022-09-23 02:25:30', 45);
INSERT INTO TEMPERATURE_READINGS (ID, TIMESTAMP, READING) VALUES ('1', '2022-09-23 02:30:30', 40);
INSERT INTO TEMPERATURE_READINGS (ID, TIMESTAMP, READING) VALUES ('1', '2022-09-23 02:35:30', 45);
INSERT INTO TEMPERATURE_READINGS (ID, TIMESTAMP, READING) VALUES ('1', '2022-09-23 02:40:30', 50);
INSERT INTO TEMPERATURE_READINGS (ID, TIMESTAMP, READING) VALUES ('1', '2022-09-23 02:45:30', 55);
INSERT INTO TEMPERATURE_READINGS (ID, TIMESTAMP, READING) VALUES ('1', '2022-09-23 02:50:30', 60);

```

Your topic and stream entries should look like this:

![Topic Entries](img_temperature_alerting_system/topic_entries.png)

![Stream Entries](img_temperature_alerting_system/stream_entries.png)

Please make sure your auto.offset.reset is set to Earliest as above.

## 4 Query detect when temperatures drops below 45 °F for a period of 10 minutes.

We need to create a query capable of detecting when the temperature drops below 45 °F for a period of 10 minutes.
However, since our sensor emits readings every 5 minutes, we need to come up with a way to detect this even
if the drop goes beyond the interval of 10 minutes. Hence why we are using hopping windows for this scenario.

```
SELECT
    ID,
    TIMESTAMPTOSTRING(WINDOWSTART, 'HH:mm:ss', 'UTC') AS START_PERIOD,
    TIMESTAMPTOSTRING(WINDOWEND, 'HH:mm:ss', 'UTC') AS END_PERIOD,
    SUM(READING)/COUNT(READING) AS AVG_READING
  FROM TEMPERATURE_READINGS
    WINDOW HOPPING (SIZE 10 MINUTES, ADVANCE BY 5 MINUTES)
  GROUP BY ID
  HAVING SUM(READING)/COUNT(READING) < 45
  EMIT CHANGES
  LIMIT 3;

```

Our query produces following output

![Select results](img_temperature_alerting_system/select_results.png)

Enter following command to list all existing streams:

### 5.1 Create Table (TRIGGERED_ALERTS)

Note that the period where the average temperature fell below 45F was exactly from 02:25 until 02:40.
This means that our alerting system is working properly. Now let’s create some continuous queries to implement this scenario.

```
CREATE TABLE TRIGGERED_ALERTS AS
    SELECT
        ID AS KEY,
        AS_VALUE(ID) AS ID,
        TIMESTAMPTOSTRING(WINDOWSTART, 'HH:mm:ss', 'UTC') AS START_PERIOD,
        TIMESTAMPTOSTRING(WINDOWEND, 'HH:mm:ss', 'UTC') AS END_PERIOD,
        SUM(READING)/COUNT(READING) AS AVG_READING
    FROM TEMPERATURE_READINGS
      WINDOW HOPPING (SIZE 10 MINUTES, ADVANCE BY 5 MINUTES)
    GROUP BY ID
    HAVING SUM(READING)/COUNT(READING) < 45;
```

### 5.2 Create stream (STREAM RAW_ALERTS)

```
CREATE STREAM RAW_ALERTS (ID VARCHAR, START_PERIOD VARCHAR, END_PERIOD VARCHAR, AVG_READING BIGINT)
    WITH (KAFKA_TOPIC = 'TRIGGERED_ALERTS',
          VALUE_FORMAT = 'JSON', PARTITIONS = 1);

```

### 5.3 Create STream (ALERTS)

```

CREATE STREAM ALERTS AS
    SELECT
        ID,
        START_PERIOD,
        END_PERIOD,
        AVG_READING
    FROM RAW_ALERTS
    WHERE ID IS NOT NULL
    PARTITION BY ID;

```

## 6. Select alerts within time window.

```

SELECT
    ID,
    START_PERIOD,
    END_PERIOD,
    AVG_READING
FROM ALERTS
EMIT CHANGES
LIMIT 3;

```

The output should look similar to:

Check underlying Kafka topic

```
PRINT ALERTS FROM BEGINNING LIMIT 3;

```

The output should look similar to:

```
Key format: JSON or KAFKA_STRING
Value format: JSON or KAFKA_STRING
rowtime: 2020/01/15 02:30:30.000 Z, key: 1, value: {"START_PERIOD":"02:25:00","END_PERIOD":"02:35:00","AVG_READING":42}, partition: 0
rowtime: 2020/01/15 02:30:30.000 Z, key: 1, value: {"START_PERIOD":"02:30:00","END_PERIOD":"02:40:00","AVG_READING":40}, partition: 0
rowtime: 2020/01/15 02:35:30.000 Z, key: 1, value: {"START_PERIOD":"02:30:00","END_PERIOD":"02:40:00","AVG_READING":42}, partition: 0
Topic printing ceased

```

END Temperature Alerting System Lab.

[Back](../README.md#Agenda) to Agenda.
