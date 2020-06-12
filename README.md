# Overview

This repository is used with a blog post that demonstrats how to setup the MongoDB Connector for Apache Kafka and use it within the Azure Event Hub environment.  It contains the following components:

| Docker Files | Description |
| --- | --- |
| docker-compose.yml | Docker Compose file that launches Kafka Connect container |
| Dockerfile-MongoConnect | Docker file that installs the MongoDB Connector for Apache Kafka |

| Python files | Description |
| --- | --- |
| create-stock-data.py | Python application that creates ficticious stock data |
| adjectives.txt | Support file for python application |
| endings.txt | Support file for python application |
| nouns.txt | Support file for python application |

| KafkaCat files | Description|
| --- | --- |
| kc.config | Config file used to make it easier to use KafkaCat with Azure Event Hubs|

See the blog post (link TBD) for a walk through on creating an Azure Event Hub, running the docker containers and configuring the MongoDB Connector for Apache Kafka.

Python application usage:

python3 create-stock-data.py <parameter_list>

| Parameter | Default Value | Description |
| --- | --- | --- |
| -s | 10 | The number of financial stock symbols to generate |
| -c | mongodb://localhost | The connection to a MongoDB cluster |
| -d | Stocks | The database to read and write to |
| -w | Source | The collection name to insert into |
| -r | Sink | The collection name to read from |

This application randomly generates a number of stocks and their initial values.  It will write new values every second for all securities into the collection defined with the “-w” parameter.  If we configured the source and sink correctly in the previous section, as the data arrives in the collection, Kafka Connect will push the data into the Azure Event Hub.  From there the Sink will notice the data in the Azure Event Hub and write it to the collection defined in the “-r” parameter.  

The output of the python application resembles the following:

WRITE: Checking MongoDB Connection
READ: Connecting to MongoDB
WRITE: Successfully connected to MongoDB
WRITE: Successfully writing stock data
{'_id': ObjectId('5ee3ba865149c02cac059014'), 'company_symbol': 'CDC', 'company_name': 'COLOSSAL DRESS CORPORATION', 'price': 91.81, 'tx_time': '2020-06-12T13:25:22Z'}
{'_id': ObjectId('5ee3ba865149c02cac059015'), 'company_symbol': 'PEP', 'company_name': 'PROUD ECLIPSE PRODUCTIONS', 'price': 52.05, 'tx_time': '2020-06-12T13:25:22Z'}
{'_id': ObjectId('5ee3ba865149c02cac059017'), 'company_symbol': 'MSV', 'company_name': 'MINUTE SEED VENTURES', 'price': 79.03, 'tx_time': '2020-06-12T13:25:22Z'}
…

The application spins up two threads, one to write the data every second and the other to open a change stream against the collection defined in the “-r” parameter.  Following the successful “writing stock data” message you will see the data being read from the change stream on the sink.  This data has made the trip from the source collection out to Azure Event Hub and back to a sink collection.
