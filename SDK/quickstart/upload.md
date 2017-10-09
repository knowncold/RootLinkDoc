---
title: 第一个实例
layout: page
category: quickstart
---
这个例子演示了如何开始使用RootLink平台，包括新建设备，新建传感器，并通过Arduino来上传数据到平台。

## 需要的硬件

- Arduino UNO
- Arduino Ethernet扩展板
- 网线
- 串口线

## 登录平台
进入[登录界面](http://118.89.28.157/login)，输入账号密码登录。

![登录](/img/register-3.png)

## 创建设备
进入[用户中心](http://118.89.28.157/dashboard/device)，能直接看到创建设备。

![](/img/upload-0.png)

按照提示，填写名称和描述即可创建设备。

![](/img/upload-1.png)

### 获得API key

![](/img/upload-2.png)

这里的`API key`在写代码的时候会经常用到，用来作为用户验证，注意不要泄露了；单击`复制`即可复制到粘贴板，也可以通过`重新生成 API`来重置`API key`。

## 设备管理

![](/img/upload-3.png)

点击中间的详情进入该设备的管理界面，左边的编辑可以修改设备的名称和描述，右边的删除可以删除该设备。

## 创建传感器

在设备管理界面单击添加传感器，选择`数值类型传感器`，填写传感器描述和传感器的单位。

![](/img/upload-4.png)

初始化的传感器没有初始值。

![](/img/upload-5.png)

## 连接硬件

如图连接Arduino和Ethernet。

## Arduino端代码
下面的代码会发送一个随机数给相应的传感器。

其中的关键就在于发送了如下的POST请求，在其他的硬件平台上构造出一样的请求即可。

> 注意，在HTTP header中需要添加`Content-Type: application/json`

```
POST /api/sensor/upload?key=ic3i9ujyb44vguja9u4703090qnkzg1u

{
  "sensorId":"07986c79-c797-47c9-8add-05e2f35a24a6",
  "value1":"17"
}
```

```c++
#include <SPI.h>
#include <Ethernet.h>
String data = "{\"sensorId\":\"07986c79-c797-47c9-8add-05e2f35a24a6\",\"value1\":\"";
String data2 = "\"}";

byte mac[] = {
  0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED
};


IPAddress ip(192, 168, 1, 63);

// fill in your Domain Name Server address here:
IPAddress myDns(192, 168, 1, 1);

// initialize the library instance:
EthernetClient client;

//char server[] = "118.89.28.157";
IPAddress server(118,89,28,157);

unsigned long lastConnectionTime = 0;             // last time you connected to the server, in milliseconds
const unsigned long postingInterval = 10L * 1000L; // delay between updates, in milliseconds
// the "L" is needed to use long type numbers

void setup() {
  // start serial port:
  Serial.begin(9600);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // give the ethernet module time to boot up:
  delay(1000);
  // start the Ethernet connection using a fixed IP address and DNS server:
  Ethernet.begin(mac, ip, myDns);
  // print the Ethernet board/shield's IP address:
  Serial.print(data);
  Serial.print("My IP address: ");
  Serial.println(Ethernet.localIP());
}

void loop() {
  // if there's incoming data from the net connection.
  // send it out the serial port.  This is for debugging
  // purposes only:
  if (client.available()) {
    char c = client.read();
    Serial.write(c);
  }

  // if ten seconds have passed since your last connection,
  // then connect again and send data:
  if (millis() - lastConnectionTime > postingInterval) {
    httpRequest();
  }

}

// this method makes a HTTP connection to the server:
void httpRequest() {
  Serial.println(random(50));
  String temp = data + random(50) + data2;
  Serial.println("##############");
  Serial.println(temp);
  // close any connection before send a new request.
  // This will free the socket on the WiFi shield
  client.stop();

  // if there's a successful connection:
  if (client.connect(server, 80)) {
    Serial.println("connecting...");
    // send the HTTP GET request:
    client.println("POST /api/sensor/upload?key=ic3i9ujyb44vguja9u4703090qnkzg1u HTTP/1.1");
    client.println("Host: 118.89.28.157");
    client.println("User-Agent: arduino-ethernet");
    client.println("Connection: close");
    client.println("Content-Type: application/json");
    client.print("Content-Length: ");
    client.println(temp.length());
    client.print("\r\n");
    client.println(temp);
    // note the time that the connection was made:
    lastConnectionTime = millis();
  } else {
    // if you couldn't make a connection:
    Serial.println("connection failed");
  }
}
```
