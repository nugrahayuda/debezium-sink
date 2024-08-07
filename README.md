# debezium-sink
Connection postgres to postgres with debezium and kafka.

In this case we will implement the following flow:
POSRTGRES >>> DEBEZIUM >>> KAFKA >>> JDBC SINK >>> POSRGRES

Here we will implement setup step by step:
1. Download docker
2. Create docker compose from docker-compose.yml
3. Make sure that docker compose runs successfully with docker-compose ps
4. Create connection to Debezium:
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" 127.0.0.1:8083/connectors/ --data "@debezium.json"
5. Prepare the database
Get in to the database source container
docker-compose exec -it postgres bash
Create the database
psql -U docker exampledb -w
CREATE TABLE student (id integer primary key, name varchar);
ALTER TABLE public.student REPLICA IDENTITY FULL;
6. Run kafka to consume the data from connection
sudo docker run --tty --network debezium_default confluentinc/cp-kafkacat kafkacat -b kafka:9092 -C -s key=s -s value=avro -r http://schema-registry:8081 -t postgres.public.student
7. Insert the data into the database
INSERT INTO student (id, name) VALUES (1, 'Jhon');
SELECT * FROM student LIMIT 100;
8. Verify the kafka offset
9. Connect to postgre-sink
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ --data "@postgres-sink.json"
10. Prepare table
psql -U docker destinations -w
CREATE TABLE student (id integer primary key, name varchar);
11. Verify the results. If successfull you can see your data from source database apear here.
