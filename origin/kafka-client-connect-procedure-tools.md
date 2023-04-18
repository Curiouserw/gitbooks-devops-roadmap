# kafka连接测试工具

# 一、简介

为了测试kafka客户端连接k8s上Kafka Bootstrap返回的信息，有一个Python脚本可显示Broker地址，并产生测试数据验证生产消费是否正常

GItHub：https://github.com/rmoff/kafka-listeners/blob/master/python/python_kafka_test_client.py



# 二、脚本使用

Python代码

```bash
from confluent_kafka.admin import AdminClient
from confluent_kafka import Consumer
from confluent_kafka import Producer
from sys import argv
from datetime import datetime

topic='test_topic'

def Produce(source_data):
    print('\n<Producing>')
    p = Producer({'bootstrap.servers': bootstrap_server})

    def delivery_report(err, msg):
        """ Called once for each message produced to indicate delivery result.
            Triggered by poll() or flush(). """
        if err is not None:
            print('❌ Message delivery failed: {}'.format(err))
        else:
            print('✅  📬  Message delivered: "{}" to {} [partition {}]'.format(msg.value().decode('utf-8'),msg.topic(), msg.partition()))

    for data in source_data:
        p.poll(0)
        p.produce(topic, data.encode('utf-8'), callback=delivery_report)

    r=p.flush(timeout=5)
    if r>0:
        print('❌ Message delivery failed ({} message(s) still remain, did we timeout sending perhaps?)\n'.format(r))

def Consume():
    print('\n<Consuming>')
    c = Consumer({
        'bootstrap.servers': bootstrap_server,
        'group.id': 'rmoff',
        'auto.offset.reset': 'earliest'
    })

    c.subscribe([topic])
    try:
        msgs = c.consume(num_messages=1,timeout=30)

        if len(msgs)==0:
            print("❌ No message(s) consumed (maybe we timed out waiting?)\n")
        else:
            for msg in msgs:
                print('✅  💌  Message received:  "{}" from topic {}\n'.format(msg.value().decode('utf-8'),msg.topic()))
    except Exception as e:
        print("❌ Consumer error: {}\n".format(e))
    c.close()

try:
    bs=argv[1]
    print('\n🥾 bootstrap server: {}'.format(bs))
    bootstrap_server=bs
except:
    # no bs X-D
    bootstrap_server='localhost:9092'
    print('⚠️  No bootstrap server defined, defaulting to {}\n'.format(bootstrap_server))


a = AdminClient({'bootstrap.servers': bootstrap_server})
try:         
    md=a.list_topics(timeout=10)
    print("""
    ✅ Connected to bootstrap server(%s) and it returned metadata for brokers as follows:
    %s
        ---------------------
        ℹ️  This step just confirms that the bootstrap connection was successful. 
        ℹ️  For the consumer to work your client will also need to be able to resolve the broker(s) returned
            in the metadata above.
        ℹ️  If the host(s) shown are not accessible from where your client is running you need to change 
            your advertised.listener configuration on the Kafka broker(s).
    """
    % (bootstrap_server,md.brokers))

    try:
        Produce(['foo / ' + datetime.now().strftime('%Y-%m-%d %H:%M:%S')])

        Consume()
    except:
        print("❌ (uncaught exception in produce/consume)")


except Exception as e:
    print("""
    ❌ Failed to connect to bootstrap server.
    
    👉 %s
    
    ℹ️  Check that Kafka is running, and that the bootstrap server you've provided (%s) is reachable from your client
    """
    % (e,bootstrap_server))
```

安装脚本依赖

```bash
python3 -m pip install confluent_kafka
```

测试命令

```bash
python3 python_kafka_test_client.py localhost:9092
```

测试输出，显示了kafka bootstrap返回给客户端的broker连接地址

```bash
🥾 bootstrap server: localhost:9092

    ✅ Connected to bootstrap server(localhost:9092) and it returned metadata for brokers as follows:
    {0: BrokerMetadata(0, curiouser:9092)}
        ---------------------
        ℹ️  This step just confirms that the bootstrap connection was successful.
        ℹ️  For the consumer to work your client will also need to be able to resolve the broker(s) returned
            in the metadata above.
        ℹ️  If the host(s) shown are not accessible from where your client is running you need to change
            your advertised.listener configuration on the Kafka broker(s).


<Producing>
✅  📬  Message delivered: "foo / 2020-12-23 18:19:24" to test_topic [partition 0]

<Consuming>
✅  💌  Message received:  "foo / 2020-12-23 18:19:24" from topic test_topic
```







# 参考：

1. https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/