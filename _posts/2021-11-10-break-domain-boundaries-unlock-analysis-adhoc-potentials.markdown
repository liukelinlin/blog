---
layout: post
title:  "Break Domain Boundaries, Unlock Potentials of Ad-hoc Data Analysis"
date:   2021-11-10 11:10:17 -0500
categories: Engineering
---
The blog designs a system and coding workflow to leverage EMR framework to break domain boundaries and unlock potentials of Ad-hoc data analysis. 

# 1. Introduction

With  
![Kafka workflow](https://liukelinlin.github.io/images/kafka-flow.jpg)

Spark is a unified analytics engine for large-scale data processing. Spark Streaming is an extension of the core Spark API that enables scalable, high-throughput, fault-tolerant stream processing of live data streams.

![Spark architecture](https://liukelinlin.github.io/images/spark-modules.jpg)

# 2. Architecture & Dataflow

The architecture is implemented as follow:

![Streaming workflow](https://liukelinlin.github.io/images/streaming-arch.jpg)

`Dataflow` can simply describe as:

**Wireshark (Tshark log) -> Kafka (Topic) -> Spark Streaming -> result (source IP, destination IP).**

After getting the live result, it is up to you where to store them. We will display the result for demo convenience.

# 3. Implementation

# 3.1 Wireshark (Tshark analyzing log)
```bash
>tshark -i en0 >>/tmp/en0.log
Capturing on 'Wi-Fi: en0'
865 
```

# 3.2 Kafka(kafka_2.12-2.4.0) setup
```bash
#start zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

#start kafka server (you can edit and copy server.properties to start multi nodes)
bin/kafka-server-start.sh config/server.properties

#create a topic (you can update parameters to support fault tolerance)
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic demo2

# import/export connector (connect-file-source.properties specifies tshark log and live import into Kafka topic "demo2": /tmp/en0.log)
bin/connect-standalone.sh config/connect-standalone.properties ../connect-file-source.properties
```

# 3.3 Spark Streaming(spark-2.4.4-bin-hadoop2.7)

Streaming code by python (pystreaming.py):

```python
#!/usr/bin/env python
# coding: utf-8
import os
import sys
import json
import logging

from pyspark import SparkConf,SparkContext
from pyspark.streaming import StreamingContext
from pyspark.streaming.kafka import KafkaUtils

os.environ['PYSPARK_SUBMIT_ARGS'] = '--packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.3.2 pyspark-shell'

if __name__=="__main__":
    logger = logging.getLogger('pyspark')
    sc = SparkContext(appName="SparkStreamingDemo2")
    sc.setLogLevel("WARN")
    ssc = StreamingContext(sc, 2)
    topic = sys.argv[1]
    kvs = KafkaUtils.createStream(ssc, "localhost:2181", "raw-event-streaming-consumer-group", {topic: 2})

    def _parse(s):
        try:
            payload = json.loads(s)["payload"]
            lst = payload.split()
            return lst[2], lst[4]
        except Exception as e:
            logger.error(e)
            return "None", "None"

    lines = kvs.map(lambda x: _parse(x[1]))
    lines.pprint()
    ssc.start()
    ssc.awaitTermination()
```

And run pystreaming.py
```bash
pip install pyspark
python pystreaming.py demo2
```

# 3.4 Finally show live streaming result.

![Streaming live result](https://liukelinlin.github.io/images/streaming-ip-result.jpg)
