##### Overview
The role of the mirror is to collect the log of the docker container and output it to the kafka message queue. 
##### How to use it
Run on each docker node,As follows:
```BASH
$ docker run -it -p 24224:24224 --user=root -v /data/out_kafka.conf:/etc/td-agent/out_kafka.conf -e FLUENTD_CONF=out_kafka.conf flunet-one
```

`NOTE`: Make sure the fluentd.conf content is set correctly.

Then, start the container log to be collected by adding the default log-driver and add the label options as follows:
```BASH
$ docker run -itd --log-driver=fluentd --log-opt tag=docker."{{.Name}}" -p 80:80 httpd
```

Now you can run the consumer command to connect the kafka cluster to view the message. As flollows:
```BASH
$ bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic docker_log --from-beginning

```

`NOTE`: 'loclahost' You can replace localhost with your zookeeper ip address

##### What ?
What is the problem, the first time to raise the issue.Thank You
