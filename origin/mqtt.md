# MQTT

# 一、简介

MQTT（Message Queuing Telemetry Transport）是一种轻量级、基于发布/订阅（publish/subscribe）模式的消息传输协议，专为带宽受限或不可靠网络环境设计，常用于物联网（IoT）设备之间的通信。

**核心概念**：

* **Broker（代理/服务器）**：负责接收、过滤、存储并转发消息的中心组件（如 Eclipse Mosquitto、EMQX、HiveMQ）。
* **Client（客户端）**：发布或订阅主题（topic）的终端或应用。
* **Topic（主题）**：消息的路由键，用于将消息分发给符合订阅的客户端，支持层级（例：`sensors/room1/temperature`）。
* **QoS（服务质量）**：消息传递保证级别，常见有 0、1、2：

  * QoS 0：最多一次（at most once），不保证到达。
  * QoS 1：至少一次（at least once），可能重复。
  * QoS 2：只有一次（exactly once），最可靠但开销最大。
* **Retained Messages（保留消息）**：代理可以保存最后一条消息，新订阅者会立即收到保留消息。
* **Last Will and Testament（遗嘱消息）**：客户端非正常断开时，broker 可发布的通知消息。
* **Keep Alive（心跳）**：用于检测客户端是否仍在线。

MQTT 的优点包括协议简单、报文小、连接开销低、支持持久会话和离线消息缓存，因而非常适合嵌入式和物联网场景。



# 二、常见的MQTT Broker

| Broker 名称              | 开源/商业      | 实现语言 | MQTT 版本         | 集群能力 | 主要特点                             | 典型使用场景                         |
| ------------------------ | -------------- | -------- | ----------------- | -------- | ------------------------------------ | ------------------------------------ |
| **Eclipse Mosquitto**    | 开源           | C        | 3.1 / 3.1.1 / 5.0 | ❌        | 轻量、稳定、资源占用低               | 嵌入式、树莓派、小规模 IoT、开发测试 |
| **EMQX**                 | 开源 + 商业版  | Erlang   | 3.x / 5.0         | ✅        | 高并发、分布式、规则引擎、桥接能力强 | 中大型 IoT 平台、私有化部署          |
| **HiveMQ**               | 商业（社区版） | Java     | 3.x / 5.0         | ✅        | 企业级稳定性、插件化、官方支持       | 金融、工业、车联网                   |
| **VerneMQ**              | 开源           | Erlang   | 3.x / 5.0         | ✅        | 高性能、水平扩展、插件机制           | 分布式 IoT 系统                      |
| **RabbitMQ (MQTT 插件)** | 开源           | Erlang   | 3.1 / 3.1.1       | ⚠️        | AMQP 为核心，支持 MQTT 插件          | 混合消息系统                         |
| **ActiveMQ Artemis**     | 开源           | Java     | 3.1 / 3.1.1       | ⚠️        | 多协议支持、Java 生态友好            | 企业消息中间件                       |

# 三、安装部署

> 下面展示常见 broker（以 Eclipse Mosquitto 为例）的安装方式，以及在 Docker 中快速运行的方法。大多数系统也可以选择其他 broker（EMQX、HiveMQ、VerneMQ 等）。

## 1、Unix/Linux

**Debian/Ubuntu**

```bash
# 安装 mosquitto broker 和客户端工具
apt install -y mosquitto mosquitto-clients
# 启动并设置为开机启动（systemd）
systemctl enable --now mosquitto
```

**CentOS/RHEL**

```bash
# 安装 EPEL 仓库（可能已存在）
yum install -y epel-release
# 安装 mosquitto
yum install -y mosquitto
# 启动服务
systemctl enable --now mosquitto
```

## 2、MacOS

```bash
# 安装 mosquitto
brew install mosquitto
brew services start mosquitto

# 设置mosquitto用户名密码文件
mosquitto_passwd -c /usr/local/etc/mosquitto/passwordfile admin
```

配置文件通常位于 `/usr/local/etc/mosquitto/mosquitto.conf`。

## 3、Docker

```bash
# 拉取镜像
docker pull eclipse-mosquitto

# 以最简单方式运行（将 1883 和可选的 WebSocket 端口 9001 映射到宿主机）
docker run -d --name mosquitto -p 1883:1883 -p 9001:9001 eclipse-mosquitto

# 挂载自定义配置（示例）
# 假设 ./mosquitto/config/mosquitto.conf 在宿主机上
docker run -d --name mosquitto \
  -p 1883:1883 -p 9001:9001 \
  -v $(pwd)/mosquitto/config:/mosquitto/config \
  -v $(pwd)/mosquitto/data:/mosquitto/data \
  -v $(pwd)/mosquitto/log:/mosquitto/log \
  eclipse-mosquitto
```

# 四、客户端

## 1、CLI

**订阅（subscribe）**：

```bash
mosquitto_sub -h localhost -t "test/topic" -v
```

**发布（publish）**：

```bash
mosquitto_pub -h localhost -t "test/topic" -m "hello mqtt" -q 1
# 参数说明：
#   -h：broker 地址（默认为 localhost）
#   -t：topic
#   -m：消息内容
#   -q：QoS（0/1/2）
#   -v：显示 topic 名称
```

## 2、GUI

常用图形化客户端：

* **[MQTTX](https://github.com/emqx/MQTTX)**：MQTTX 是EMQ开源的跨平台MQTT 5.0客户端工具，可在macOS、Linux和 Windows上运行，并支持格式化 MQTT Payload
* **[MQTT Explorer](https://github.com/thomasnordquist/MQTT-Explorer)**：一款现代、树状显示 topic 的开源客户端，适合调试和分析消息流。
* **MQTT.fx**：传统的 MQTT GUI 客户端，支持连接配置、订阅和发布。
* **HiveMQ Web Client**：在线/自托管的 Web 客户端，便于快速验证连接。

## 3、Go

Go 语言常用客户端库：`github.com/eclipse/paho.mqtt.golang`

```go
package main

import (
    "fmt"
    mqtt "github.com/eclipse/paho.mqtt.golang"
    "time"
)

func main() {
    opts := mqtt.NewClientOptions().AddBroker("tcp://localhost:1883").SetClientID("go-sample-client")
    client := mqtt.NewClient(opts)
    if token := client.Connect(); token.Wait() && token.Error() != nil {
        panic(token.Error())
    }

    // 订阅
    client.Subscribe("test/topic", 1, func(client mqtt.Client, msg mqtt.Message) {
        fmt.Printf("Received message: %s from topic: %s\n", msg.Payload(), msg.Topic())
    })

    // 发布
    client.Publish("test/topic", 1, false, "Hello from Go!")

    // 等待，演示用
    time.Sleep(2 * time.Second)
    client.Disconnect(250)
}
```

## 4、Java

Java 常用客户端：Eclipse Paho Java Client（`org.eclipse.paho.client.mqttv3`）

```java
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;

public class JavaMqttSample {
    public static void main(String[] args) throws MqttException {
        String broker = "tcp://localhost:1883";
        String clientId = "java-sample-client";

        MqttClient client = new MqttClient(broker, clientId);
        MqttConnectOptions connOpts = new MqttConnectOptions();
        connOpts.setCleanSession(true);
        client.connect(connOpts);

        client.setCallback(new MqttCallback() {
            public void connectionLost(Throwable cause) { }
            public void messageArrived(String topic, MqttMessage message) throws Exception {
                System.out.println("Received: " + new String(message.getPayload()));
            }
            public void deliveryComplete(IMqttDeliveryToken token) { }
        });

        client.subscribe("test/topic", 1);
        MqttMessage message = new MqttMessage("Hello from Java".getBytes());
        message.setQos(1);
        client.publish("test/topic", message);

        // 断开前等待
        client.disconnect();
    }
}
```

## 5、JavaScript

JavaScript/Node.js 常用库：`mqtt`（npm 包名 `mqtt`），浏览器端也可通过 WebSocket 连接到 broker（如果 broker 支持 WebSocket）

```javascript
const mqtt = require('mqtt')
const client  = mqtt.connect('mqtt://localhost:1883')

client.on('connect', function () {
  client.subscribe('test/topic', { qos: 1 }, function (err) {
    if (!err) {
      client.publish('test/topic', 'Hello from Node.js', { qos: 1 })
    }
  })
})

client.on('message', function (topic, message) {
  // message 是 Buffer
  console.log(topic, message.toString())
})
```

**浏览器端**：

* 使用 MQTT over WebSocket：broker 需开启 WebSocket 端口（如 `ws://broker:9001`），然后在浏览器中用 `mqtt` 客户端或其他 WebSocket 客户端进行连接。



# 附录

## 1、注意事项

1. **认证与授权**：生产环境建议启用用户名/密码或者使用 TLS + 客户端证书实现身份校验，并配合 broker 的 ACL（访问控制列表）限制主题访问。
2. **加密**：使用 TLS（`mqtts://` 或 `wss://`）来保护消息在传输中的机密性，尤其是在公网上部署时。
3. **可伸缩性**：单机 broker 适用于小规模部署；大规模场景可选择支持集群的 broker（EMQX、HiveMQ、VerneMQ）并结合负载均衡与持久化策略。
4. **性能优化**：合理选择 QoS，控制保留消息与持久会话的使用；监控连接数和消息吞吐，使用持久化存储避免数据丢失。
5. **调试技巧**：优先使用 CLI（`mosquitto_pub`/`mosquitto_sub`）进行快速连通性测试；在开发阶段可用 MQTT Explorer 观察主题树与消息流。
