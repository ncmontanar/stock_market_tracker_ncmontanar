## Derarrer le projet 

1.1. Creer l'instance (VM) EC2 avec Amazon Linux 2022 - stock_market_tracker

1.2. Creer une clé d'autentification et la stocker dans le dossier du projet

1.3. Se connecter à l'instance par : a. ssh  ou b. directement

 - ssh : en VS, ubicarse en el archivo del projecto y lanzar la cmd 
'ssh -i "stock_market_tracker_llave.pem" ec2-user@ec2-15-237-139-160.eu-west-3.compute.amazonaws.com'

1.4 **pour parametrer les inbound rules**
    *SSH -> Custom 0.0.0.0
    SSH -> myIp  xxxxxxx
    alltrafic -> Custom 0.0.0.0*

---
### Setup the EC2 (Zookepeer - gestion de configuration pour systèmes distribués)
2) 
En la instancia :  ATTENTION A LA VERSION DE KAFKA
2.1 Télécharger Kafka : 
    `wget https://archive.apache.org/dist/kafka/3.3.1/kafka_2.13-3.3.1.tgz`
---
2.2 unzip
    `tar -xvf kafka_2.13-3.3.1.tgz`
---
2.3. Télécharger java
```
    java -version
    sudo yum install java-1.8.0-openjdk
    java -version
```
aller à 
    `cd kafka_2.13-3.3.1`
---
2.4 démarrer Zookeeper (in kafka_2.12-3.3.1)
    `bin/zookeeper-server-start.sh config/zookeeper.properties`

### Setup the EC2 == Start kafka server ( Creation du topic + producer)
3) 
3.1) Dans une nouvelle terminal de l'instance elargir la taille de la mémoire
    `export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M" `

aller à :
    `cd kafka_2.13-3.3.1`
puis (3.2)
    `bin/kafka-server-start.sh config/server.properties`

#### on peut pas utiliser ce serveur car is pointing to private server , change server.properties so that it can run in public IP  (pleintext://...) for a public server (our public IP)
Donc, stop les 2 sessions, and update the file *config/server.properties* qui se trouve sous *kafka_2.13-3.3.1* ; puis sudo nano config/server.properties; et change la ligne ADVERTISED_LISTENERS=PLAINTEXT... pour la  public ip  de notre  EC2 instance

puis relance Zookeeper(2.4) and relance kafka-server (3.2)

__Verifier les inbound rules__
    *SSH -> Custom 0.0.0.0
    alltrafic -> myIp  xxxxxxx
    alltrafic -> anywhere 0.0.0.0*

#### Et create the topic and producer in a new window EC2 instance (keep to connect again)
***ejem***
(bin/kafka-topics.sh --create --topic myname --bootstrap-server {Public IP of your EC2 Instance:9092} --replication-factor 1 --partitions 1)
***ejem2*** 
    `bin/kafka-topics.sh --create --topic prodc_test3 --bootstrap-server 51.44.15.123:9092 --replication-factor 1 --partitions 1`

ignore the warning
3.2)then Start the Producer: (in the same console where it was created topic (3.1) attention avec le meme nom)

***ejem*** 
(bin/kafka-console-producer.sh --topic myname --bootstrap-server {Put the Public IP of your EC2 Instance:9092} )
***ejem***
    `bin/kafka-console-producer.sh --topic prodc_test3 --bootstrap-server 51.44.15.123:9092`

---
5) 
4.1) Duplicate the session & enter in a new console: consumer
 Start Consumer:
    `cd kafka_2.12-3.3.1`
***ejem***    
(bin/kafka-console-consumer.sh --topic myname --bootstrap-server {Public IP of your EC2 Instance:9092})
***ejem***
`bin/kafka-console-consumer.sh --topic prodc_test3 --bootstrap-server 51.44.15.123:9092`

---
5) ### creer new jupy notebook : kaf_producer

- 5.2. faire les pips
    - `pip install kafka-python`
    - `pip install pandas`
- 5.3 faire les imports 
    ```
    import pandas as pd
    from kafka import KafkaProducer
    from kafka import KafkaConsumer
    from time import sleep
    from json import dumps
    import json
    ```
- 5.4 # creer l'object for the kafka_producer (avec mon Public IPv4 address)
- 5.5 create à dock object -> to start sent data in value={}
--> 6 
5.6 put it inside a loop (sleep va après)
5.7 envoie la donnée vers le consommer

---
6) ### 6.1 creer new jupy notebook : kaf_producer

6.2. faire les pips
    `pip install s3fs`
6.3 faire les imports (35:15)
    ```
    from kafka import KafkaConsumer
    from time import sleep
    from json import dumps,loads
    import json
    from s3fs import S3FileSystem
    ```
6.4 creer object for the kafka_consumer
---
 -> creation du bucket s3 pour stocker les resultats
 *stock-mkt-tracker-bukt-ncmontanar*

-> en IAM
    - create user and allow him AdministratorAccess (or chose the user who has it )
    - download the secret acces key (or recuperate the Access keys  )
    - in the free console `aws configure` and log it
    - that allows to send data from my local to S3
---
6.5 flush the producer
6.6 run the *while producer* and wait 5 secs 
6.7 run la boucle for *consumer dump* 
6.8 update the bucket s3
---
### go to glue crawler
7.1 set to crawler propierties (stock_market_kefka_project)
7.2 add a datasource
7.3 in S3 path, chose ous S3 bucket *stock-mkt-tracker-bukt-ncmontanar/stock_market*
7.4  IAM role : un role who has AWSGlueadmin acces
7.5 create a database : *stock_market_dbb*
7.6 go foward next = create crawler
7.7 run crawler until succes
---
### go to Athena
8.1 chose our dbb  *stock_market_dbb*
8.2 run a sample query
8.3 
```
--SELECT * FROM "stock_market_dbb"."stock_mkt_tracker_bukt_ncmontanar" limit 10;
--SELECT max(date) FROM "stock_market_dbb"."stock_mkt_tracker_bukt_ncmontanar" ;
-- SELECT count(*) FROM "stock_market_dbb"."stock_mkt_tracker_bukt_ncmontanar" ;
SELECT index, date
FROM stock_market_dbb.stock_mkt_tracker_bukt_ncmontanar
WHERE index = 'NYA';
```


**bold text** / __bold text__
*cat's meow*
***Bold and Italic***
##### Heading level 5
###### Heading level 5
> Blockquotes
To denote a word or phrase as code, enclose it in backticks `` : `nano`
---  Horizontal Rules
Fenced Code Blocks :
```
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```
==Highlight==