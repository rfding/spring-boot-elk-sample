# Spring Boot ELK Logging Sample
#### Centralized Logging
This app is configured to send logs to ELK (Elasticsearch, Logstash, Kibana), which should be running on a Docker stack.

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

### Configuration Options

This Spring Boot application uses Logback to send logs to Logstash. Loggers can be configured in `logback-spring.xml` and variables can be read in from `application.properties`.

| Property      | Example           | Description  |
| ------------- |:-------------:| -----|
| `logstash.host`| `logstash:5000` | The Logstash port to send logs to. This must match the exposed tcp port in the Logstash configuration file. See below for sample Logstash conf file.
| `logger.name`| `hello` | The name of the logger. Usually matches the name of the package or file that the logger corresponds to

New properties can be created in `application.properties` then read into `logback-spring.xml` by using the `<springProperty>` tag.

#### Sample Logstash Configuration File

<details><summary>Click to expand</summary>

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

</details>