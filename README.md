# Confluent Cloud ksqlDB Hands-On Workshop 2022 Wiesn Edition

This hands-on workshop walks you through the initial steps of using ksqlDB as an event stream processing engine in Confluent Cloud. Its exercises cover several popular event stream processing patterns and demonstrate how they can be implemented as data pipelines leveraging the ksql language and the Kafka data in motion infrastructure. This guide is not for people that already have experience with ksqlDB and that are looking for an in-depth learning resource, nor does it cover any automated development, testing or deployment mechanisms and tooling for ksqlDB based data pipelines.

The Confluent Cloud ksqlDB Hands-On Workshop labs and exercises are optimized for learning, which means taking a fully guided step-by-step approach using the Confluent Cloud Wen UI, to ensure you understand each task required to develop and run an end-to-end data pipeline while only requiring access to a web-browser.

> The results of this workshop should not be viewed as production ready, as they are totally focussed on learning by getting started with ksqlDB in Confluent Cloud.

This tutorial assumes you have access to [Confluent Cloud](https://confluent.cloud). While Confuent Cloud is used for the lowest barrier to fullfill the basic infrastructure requirements the lessons learned in this workshop can be applied to other deployment models Confluent Platform, as well as with Apache Kafka and a stand-alone [ksqlDB engine](https://ksqldb.io).

## Agenda

### 0. Environment Setup

- [Confluent Cloud environment setup](environment_setup/envirnment_setup_confluent_cloud.md)

### 1. Lab Financial Services

The Financial Services Lab, walks you through two exercise that address common event stream processing patterns with ksqlDB that will be applied in the context of the following financial industry use-cases:

- [Payment Status Check](lab_finanzdaten/FinancialServices_Payment_Status_Check.md)
- [Customer Loyalty Program](lab_finanzdaten/FinancialServices_Customer_Loyalty_Program.md)

### 2. Lab Retail and Logistics

Retail and Logistics Lab, walks you through two exercise that address common event stream processing patterns with ksqlDB that will be applied in the context of the following retail use-cases:

- [Masking PII Data](lab_logistik/masking.md)
- [Track & Trace](lab_logistik/Track_and_Trace.md)

### 3. Lab Internet of Things (IoT)

The Internet of Things Lab, walks you through two exercise that address common event stream processing patterns with ksqlDB that will be applied in the context of the following IoT use-cases:

- [Sensor Reading Time Series Normalisation](lab_iot/Pump_Stream_Processing.md)
- [IoT Anomaly Detection](lab_iot/Temperature_Alerting_System.md)
