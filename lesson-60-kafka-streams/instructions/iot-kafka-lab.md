# IOT Kafka Stream Solution

In this example, we process real-world vehicle IoT data. Our data is in file `vehicle_1_1000.tsv`. This file contains first 1000 rows of the vehicle sensor data representing car movements..

The data stored is in a tab-separated file. The values represents observed position of the tracked vehicles

### Data Schema

The data fields are:

- Col 1: Device ID (unique for each vehicle)
- Col 2: Time of the observation (in UTC)
- Col 3: Speed of the vehicle
- Col 4: The compass direction of the vehicle
- Col 5: The longitude of the GPS coordinates
- Col 6: The latitude of the GPS coordinates

#### Accuracy of GPS Coordinates

For GPS coordinates, about 100 meters accuracy is roughly coordinates rounded to 3 decimal places.

See [GIS Accuracy](https://gis.stackexchange.com/questions/8650/measuring-accuracy-of-latitude-and-longitude) for a detailed explanation.

We'll use a trick using geohash.
Geohash is an open source algorithm that splits the world into tiles.
You can read about geohashes [here](https://en.wikipedia.org/wiki/Geohash)

## The Goal

The goal is to count the number of times a car is observed parked at the same location with accuracy of about 100 meters.
We'll use Kafka Streams to do so.

## The lab

In this lab we'll simply run a possible solution.

The solution contains two projects.

* `gps-pump`
  * This project contains a program that can publish GPS coordinates
  * The program is written in Java and it takes an input file (in TSV format) as an argument. The program repeatedly pushes the content of the TSV file to a topic `gps-locations`
* `processor`
  * This is another Java project that uses Kafka Streams to process the incoming GPS coordinates and find the places where vehicles are most likely to park.

## Steps of the exercise

### Start Kafka

Unless you've already done so, follow start Kafka through docker (the instructions for this is in previous exercises).

### Create the topics

```
$ docker-compose exec kafka /opt/kafka/bin/kafka-console-producer.sh --broker-list kafka:9092 --topic gps-locations
$ docker-compose exec kafka /opt/kafka/bin/kafka-console-producer.sh --broker-list kafka:9092 --topic frequent-parking
```

### Build the `gps-pump` project

Go to the `instructions/iot-kafka-solution/gps-pump` directory and build the project:

```sh
mvn clean package
```

### Run the GPS pump

Next, run the project pointing passing the test data to the program:

```sh
target/gps-pump ../vehicle_1_1000.tsv
```

### Build the processor

In a new terminal, enter the directory `instructions/iot-kafka-solutions/processor` and build the project:

```sh
mvn clean package
```

### Run the processor

```sh
target/processor
```

### Where are we?

We now have a program (`gps-pump`) producing GPS locations into a topic (`gps-locations`). We also have a program processing these locations and producing events into another topic (`frequent-parking`) with the most likely places the various vehicles will park.

### See the result

To see the result, we'll have to start a console consumer that can read the output on the frequent parking parking.

Open another terminal and enter the docker directory. In the docker directory run:

```sh
$ docker-compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic frequent-parking --from-beginning --property print.key=true
```
