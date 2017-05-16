# Marathon-lb logs bundle

Docker image derivated from the official marathon-lb with a prometheus exporter and a kafka log shipper.

### Description

Adds useful metrics and log shipper helpers to stock marathon-lb.

This project produce an automated built docker image [vixns/mlb](https://hub.docker.com/r/vixns/mlb/), always in sync with the official [mesosphere/marathon-lb](https://hub.docker.com/r/mesosphere/marathon-lb/) docker image.

### Contents

- latest stable marathon-lb with no custom changes.
- a prometheus exporter
- syslog + omkafka
- custom mlb templates


## Requirements

This bundle can run without extra dependencies, but there is no point to use it unless you can ship your logs and metrics.

You have to provide at least one of : 

- [prometheus](https://github.com/prometheus/prometheus)

This example assumes you run this bundle as `/mlb` app in marathon, adjust the names to your setup.

```
  - job_name: 'proxy'
    scrape_interval: 5s
    dns_sd_configs:
    - names: ['mlb.marathon.mesos']
      type: 'A'
      port: 9101	
```

- kafka 
You can use any kafka broker, our recommandation is to use the mesos scheduler. 
We maintain and use in production a custom automated built docker image [vixns/kafka-mesos](https://hub.docker.com/r/vixns/kafka-mesos/).

For performance and interoperability reasons, haproxy logs are sent as is to kafka, without any formatting. Parsing operations have to be made on the kafka consumers side.


### Usage

Marathon-lb is typicaly run by marathon, but can also be launched manually

	docker run --rm \
	-p 9090:9090 \
	-p 8080:80 \
	-e PORTS=9090 \
	-e KAFKA_TOPIC=mlb-logs \
	-e KAFKA_BROKERS="kafka-0.kafka.mesos:30092,kafka-1.kafka.mesos:30092" \
	vixns/mlb

The kafka shipper can be disabled, remove the `KAFKA_BROKERS` env var if you just need the prometheus exporter.

Your logs are now shipped to kafka, you can follow them with kafkacat 

	kafkacat -b kafka-0.kafka.mesos:30092,kafka-1.kafka.mesos:30092 -G test mlb-logs

### logs export

You can export you logs from kafka for many different uses.

#### EFK

One common use is to export them to elastic and visualise them with kibana.
We maintain a fluentd docker image with the required plugins, [vixns/fluentd](https://hub.docker.com/r/vixns/fluentd/).
Sample configuration is provided in the `examples/fluentd` folder.

#### prometheus

The bundle prometheus exporter is for haproxy metrics. These metrics are quite useful from a cluster operations perspective, but lack some useful informations, like detailed status code, referers ...

We maintain an unofficial prometheus-haproxy-log-exporter docker image with kafka input support [vixns/prometheus-haproxy-log-exporter](https://hub.docker.com/r/vixns/prometheus-haproxy-log-exporter/).







