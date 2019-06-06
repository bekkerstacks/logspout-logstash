# logspout-logstash
Logstash Support for Logspout

## Description

Logspout is a log router for docker containers. Using it in conjunction with Logstash, gives you:

`Docker -> Logspout -> Logstash -> Elasticsearch <- Kibana`

## Requirements

Example stack if you don't have an ELK Stack running:
- https://github.com/bekkerstacks/elk

## Configuration

Logspout configuration:

```
  logspout:
    image: ruanbekker/logspout-logstash:v3.2.4
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - ROUTE_URIS=logstash+tcp://logstash:5000
    networks:
      - esnet
    deploy:
      mode: global
```

Logstash configuration:

```
input {
  udp {
    port => 5000
    codec => json
  }
  tcp {
    port => 5000
    codec => json
    type => dockerlog
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
  }
  stdout { }
}
```

Application configuration:

```
  webapp:
    image: nginx
    networks:
      - esnet
    environment:
      - LOGSTASH_TAGS=docker,production
      - LOGSTASH_FIELDS=service_name={{.Service.Name}},task.id={{.Task.ID}}
      - DOCKER_LABELS=""
    ports:
      - 8082:80
    labels:
      com.example.app.description: "value 1"
    deploy:
      labels:
        com.example.deploy.description.: "value 2"
```

## More Info:

- https://github.com/thojkooi/logspout-logstash
- https://github.com/looplab/logspout-logstash
