# Pulsar WebSocket API

# 一、简介

Pulsar WebSocket API 提供了一种使用没有官方客户端库的语言与 Pulsar 交互的简单方法。通过 WebSocket，您可以发布和使用消息并使用客户端功能矩阵页面上提供的功能。

官方文档：https://pulsar.apache.org/docs/client-libraries-websocket/

# 二、配置

## 1、在Broker的8080端口开起

`conf/broker.conf`

```bash
webSocketServiceEnabled=true
```

## 2、作为独立组件启用

`conf/websocket.conf` 

```bash
configurationMetadataStoreUrl=zk1:2181,zk2:2181,zk3:2181
webServicePort=8080
clusterName=my-cluster

# 如果要开起TLS，需要设置一下参数
tlsEnabled=true
tlsAllowInsecureConnection=false
tlsCertificateFilePath=/path/to/client-websocket.cert.pem
tlsKeyFilePath=/path/to/client-websocket.key-pk8.pem
tlsTrustCertsFilePath=/path/to/ca.cert.pem
```

```bash
bin/pulsar-daemon start websocket
```

# 三、Websocket工具

- Chrome插件：PieSocket WebSocket Tester
- 网页工具：http://wstool.js.org/
- Python：`pip3 install websocket-client`
  - 文档：https://pulsar.apache.org/docs/client-libraries-websocket/#python

# 四、API Endpoint

## 1、生产者Endpoint

### 请求URL

```bash
ws://broker-service-url:8080/ws/v2/producer/persistent/:tenant/:namespace/:topic?参数1=值&参数2=值
```

| 参数                      | 类型    | 必须 | 描述                                                         |
| ------------------------- | ------- | ---- | ------------------------------------------------------------ |
| `sendTimeoutMillis`       | long    | no   | Send timeout (默认: 30s)                                     |
| `batchingEnabled`         | boolean | no   | Enable batching of messages (默认: false)                    |
| `batchingMaxMessages`     | int     | no   | 批处理中允许的最大消息数 (默认: 1000)                        |
| `maxPendingMessages`      | int     | no   | Set the max size of the internal-queue holding the messages (默认: 1000) |
| `batchingMaxPublishDelay` | long    | no   | Time period within which the messages will be batched (默认: 10ms) |
| `messageRoutingMode`      | string  | no   | Message [routing mode](https://pulsar.apache.org/api/client/index.html?org/apache/pulsar/client/api/ProducerConfiguration.MessageRoutingMode.html) for the partitioned producer: `SinglePartition`, `RoundRobinPartition` |
| `compressionType`         | string  | no   | Compression [type](https://pulsar.apache.org/api/client/index.html?org/apache/pulsar/client/api/CompressionType.html): `LZ4`, `ZLIB` |
| `producerName`            | string  | no   | Specify the name for the producer. Pulsar will enforce only one producer with same name can be publishing on a topic |
| `initialSequenceId`       | long    | no   | Set the baseline for the sequence ids for messages published by the producer. |
| `hashingScheme`           | string  | no   | [Hashing function](https://pulsar.apache.org/api/client/org/apache/pulsar/client/api/ProducerConfiguration.HashingScheme.html) to use when publishing on a partitioned topic: `JavaStringHash`, `Murmur3_32Hash` |
| `token`                   | string  | no   | Authentication token, this is used for the browser javascript client |

### 消息体样本

```json
{
  "payload": "SGVsbG8gV29ybGQ=",
  "properties": {"key1": "value1", "key2": "value2"},
  "context": "1"
}
```

### 消息体属性

| 属性                  | 类型            | 必须 | 描述                                                         |
| --------------------- | --------------- | ---- | ------------------------------------------------------------ |
| `payload`             | string          | yes  | Base-64 encoded payload                                      |
| `properties`          | key-value pairs | no   | Application-defined properties                               |
| `context`             | string          | no   | Application-defined request identifier                       |
| `key`                 | string          | no   | For partitioned topics, decides which partition to use       |
| `replicationClusters` | array           | no   | Restrict replication to this list of [clusters](https://pulsar.apache.org/docs/reference-terminology#cluster), specified by name |

## 2、消费者Endpoint

### 请求URL

```bash
ws://broker-service-url:8080/ws/v2/consumer/persistent/:tenant/:namespace/:topic/:subscription?参数1=值&参数2=值
```

| Key                          | Type    | Required? | Explanation                                                  |
| ---------------------------- | ------- | --------- | ------------------------------------------------------------ |
| `ackTimeoutMillis`           | long    | no        | 设置未确认消息的超时时间 (默认: 0)                           |
| `subscriptionType`           | string  | no        | 订阅类型: `Exclusive`, `Failover`, `Shared`, `Key_Shared`    |
| `receiverQueueSize`          | int     | no        | 设置消费者接收队列的大小 (默认: 1000)                        |
| `consumerName`               | string  | no        | 消费者名字                                                   |
| `priorityLevel`              | int     | no        | 设置消费者定义优先级                                         |
| `maxRedeliverCount`          | int     | no        | Define a [maxRedeliverCount](https://pulsar.apache.org/api/client/org/apache/pulsar/client/api/ConsumerBuilder.html#deadLetterPolicy-org.apache.pulsar.client.api.DeadLetterPolicy-) for the consumer (默认: 0). Activates [Dead Letter Topic](https://github.com/apache/pulsar/wiki/PIP-22%3A-Pulsar-Dead-Letter-Topic) feature. |
| `deadLetterTopic`            | string  | no        | Define a [deadLetterTopic](https://pulsar.apache.org/api/client/org/apache/pulsar/client/api/ConsumerBuilder.html#deadLetterPolicy-org.apache.pulsar.client.api.DeadLetterPolicy-) for the consumer (默认: {topic}-{subscription}-DLQ). Activates [Dead Letter Topic](https://github.com/apache/pulsar/wiki/PIP-22%3A-Pulsar-Dead-Letter-Topic) feature. |
| `pullMode`                   | boolean | no        | 是否开起pull模式 (默认: false)                               |
| `negativeAckRedeliveryDelay` | int     | no        | When a message is negatively acknowledged, the delay time before the message is redelivered (in milliseconds). 默认: 60000. |
| `token`                      | string  | no        | Authentication token, this is used for the browser javascript client |

### 消息体样本

```json

{
  "messageId": "CAMQADAA",
  "payload": "hvXcJvHW7kOSrUn17P2q71RA5SdiXwZBqw==",
  "properties": {},
  "publishTime": "2021-10-29T16:01:38.967-07:00",
  "redeliveryCount": 0,
  "encryptionContext": {
    "keys": {
      "client-rsa.pem": {
        "keyValue": "jEuwS+PeUzmCo7IfLNxqoj4h7txbLjCQjkwpaw5AWJfZ2xoIdMkOuWDkOsqgFmWwxiecakS6GOZHs94x3sxzKHQx9Oe1jpwBg2e7L4fd26pp+WmAiLm/ArZJo6JotTeFSvKO3u/yQtGTZojDDQxiqFOQ1ZbMdtMZA8DpSMuq+Zx7PqLo43UdW1+krjQfE5WD+y+qE3LJQfwyVDnXxoRtqWLpVsAROlN2LxaMbaftv5HckoejJoB4xpf/dPOUqhnRstwQHf6klKT5iNhjsY4usACt78uILT0pEPd14h8wEBidBz/vAlC/zVMEqiDVzgNS7dqEYS4iHbf7cnWVCn3Hxw==",
        "metadata": {}
      }
    },
    "param": "Tfu1PxVm6S9D3+Hk",
    "compressionType": "NONE",
    "uncompressedMessageSize": 0,
    "batchSize": {
      "empty": false,
      "present": true
    }
  }
}

```

### 消息体属性

- 基础属性

  | Key                       | Type              | Required? | Explanation                                                  |
  | ------------------------- | ----------------- | --------- | ------------------------------------------------------------ |
  | `messageId`               | string            | yes       | 消息 ID                                                      |
  | `payload`                 | string            | yes       | Base-64 encoded payload                                      |
  | `publishTime`             | string            | yes       | 推送时间戳                                                   |
  | `redeliveryCount`         | number            | yes       | Number of times this message was already delivered           |
  | `properties`              | key-value pairs   | no        | Application-defined properties                               |
  | `key`                     | string            | no        | Original routing key set by producer                         |
  | `encryptionContext`       | EncryptionContext | no        | Encryption context that consumers can use to decrypt received messages |
  | `param`                   | string            | no        | Initialization vector for cipher (Base64 encoding)           |
  | `batchSize`               | string            | no        | Number of entries in a message (if it is a batch message)    |
  | `uncompressedMessageSize` | string            | no        | Message size before compression                              |
  | `compressionType`         | string            | no        | Algorithm used to compress the message payload               |

- `encryptionContext` related parameter

  | Key    | Type                    | Required? | Explanation                                                  |
  | ------ | ----------------------- | --------- | ------------------------------------------------------------ |
  | `keys` | key-EncryptionKey pairs | yes       | Key in `key-EncryptionKey` pairs is an encryption key name. Value in `key-EncryptionKey` pairs is an encryption key object. |

- `encryptionKey` related parameters

  | Key        | Type            | Required? | Explanation                      |
  | ---------- | --------------- | --------- | -------------------------------- |
  | `keyValue` | string          | yes       | Encryption key (Base64 encoding) |
  | `metadata` | key-value pairs | no        | Application-defined metadata     |