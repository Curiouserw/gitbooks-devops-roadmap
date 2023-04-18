# Python Pulsar Client

# 一、Python Client

```bash
pip3 install pulsar-client
```

`pulsar-client.py`

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-

import pulsar,sys,socket
def consume(client,topic):
    consumer = client.subscribe(topic,subscription_name=(socket.gethostbyname(socket.gethostname())))
    while True:
        msg = consumer.receive()
        print("Received message: '%s'" % msg.data())
        consumer.acknowledge(msg)
    client.close()
def produce(client,topic,msg):
    producer = client.create_producer(topic)
    producer.send((msg).encode('utf-8'))
    client.close()
def main(args):
    client = pulsar.Client('pulsar://localhost:6650')
    if args[1] == "consume":
        consume(client,args[2])
    elif args[1] == "produce":
        produce(client,args[2],args[3])

if __name__ == "__main__":
    main(sys.argv)
```



## 1、生产数据

```bash
python3 ./pulsar-client.py produce test "Hello Pulsar , I'm python client"
```

## 2、消费数据

```bash
python3 ./pulsar-client.py consume test
```
