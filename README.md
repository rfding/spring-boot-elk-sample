# Spring Boot ELK Logging Sample
#### Centralized Logging
This sample Spring Boot app is configured to send logs to ELK (Elasticsearch, Logstash, Kibana), which should be running on a Docker stack before running the app. The app sends logs to stdout, to files, and to Logstash, which parses the logs and sends them to Elasticsearch.

## File Overview
The dependencies that are vital to sending logs to Logstash are the Logstash Logback encoder and Logback classic.

```
<dependencies>
	<dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>5.3</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
</dependencies>
```

The loggers are configured in `logback-spring.xml`. The logger that handles logging to Logstash is named `STASH`. More details about logger configuration are located [here](#configuration-options).
```
<appender name="STASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <destination>${LOGSTASH_HOST}</destination>
    <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
</appender>
```

## How to use

Build the Docker image
```
> ./mvnw package
> docker build -t test/test . 
```

Run the docker stack:
```
> docker stack deploy -c docker-stack.yml sample
Creating service sample_sample
```

Open Kibana to view the logs. They should be indexed beginning with `logstash-`.

## Configuration Options

This Spring Boot application uses Logback to send logs to Logstash. Loggers can be configured in `logback-spring.xml` and variables can be read in from `application.properties`. The logger in the sample is the most basic version. For more configuration options, check [here](https://github.com/logstash/logstash-logback-encoder#tcp-appenders).

| Property      | Example           | Description  |
| ------------- |:-------------:| -----|
| `logstash.host`| `logstash:5000` | The Logstash port to send logs to. This must match the exposed tcp port in the Logstash configuration file. See below for sample Logstash conf file.
| `logger.name`| `hello` | The name of the logger. Usually matches the name of the package or file that the logger corresponds to.

New properties can be created in `application.properties` then read into `logback-spring.xml` by using the `<springProperty>` tag.

### Sample Logstash Configuration File

`elk/src/templates/configs/logstash/logstash.conf`:

```
input {
	tcp {
		port => 5000
		codec => json_lines
	}
}

output {
	elasticsearch {
		hosts => "elasticsearch:9200"
	}
}
```

**Note:** `codec => json_lines` is necessary for Logstash to be able to parse the JSON logs correctly.
