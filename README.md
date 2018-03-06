## KSQL Workshop

Instructions for a brief KSQL hands-on workshop. Assume Linux or Mac. If using Windows, please use the provided Ubuntu VM.

1. Set up the Confluent KSQL developer preview.

  * `git clone https://github.com/confluentinc/ksql`
  
  * `cd ksql`
  
  * `git checkout v0.5`
  
  * `mvn clean compile install -DskipTests`
  
  * Let Maven do its thing.
  
  * After Confluent Platform is installed and running, run KSQL with `bin/ksql-cli local`
  
2. Install the Confluent Platform.

  * Apache Kafka plus some other open source components
  * Expand the tarball
  * Add `$CONFLUENT_HOME/bin` to your path
  * (You don't literally need a `$CONFLUENT_HOME` environment variable, but it's nice and clean)
  
3. Get the data generator.

  * `git clone https://github.com/tlberglund/streams-movie-demo`
  
  * Bootstrap Gradle by running `gradlew build`

4. Add the following lines to the end of your `$CONFLUENT_HOME/etc/kafka/server.properties` file
```
port = 9092
advertised.host.name = localhost
```
5. Start the Confluent Platform

  * Type `confulent start` at any old terminal

6. Calculate average movie ratings using JSON

  * Load `data/movies-json.js` into a topic called `movies-raw` with `kafka-console-producer --broker-list localhost:9092 --topic movies_raw`
  * `CREATE STREAM movies-raw  ... (VALUE_FORMAT='JSON', KAFKA_TOPIC='movies-raw')`
  * Repartition the stream with `CREATE STREAM movies_rekeyed AS SELECT * FROM movies_raw PARTITION BY movie_id;`
  * Make movies into a lookup table with `CREATE TABLE movies_ref ... WITH (VALUE_FORMAT='JSON', KAFKA_TOPIC='movies_rekeyed', KEY='movie_id');`
  * Start the movie rating streamer with `gradlew streamJsonRatings`
  * Make a stream for the ratings topic `CREATE STREAM ratings (movie_id BIGINT, rating DOUBLE) WITH (VALUE_FORMAT = 'JSON', KAFKA_TOPIC='ratings');`
  * Write your own `SELECT` statement that joins `movies_ref` to `ratings`
  * Write your own `CREATE TABLE AS SELECT` statement that joins `movies_ref` to `ratings` and projects only title, release year, and _average rating_ (which you must calculate)
  
