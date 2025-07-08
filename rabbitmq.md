# learning-materials  

## rabbitmq学习（C++，centos）  
### 在linux上安装rabbitmq服务  
#### 进行一些前置操作
```bash
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc'  
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key'    
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key'
```
#### 安装rabbitmq-server并启动
```bash
dnf update -y  
dnf install -y logrotate  
dnf install -y erlang rabbitmq-server  
systemctl enable rabbitmq-server
```
#### 安装管理插件并重启服务
```bash
sudo rabbitmq-plugins enable rabbitmq_management  
sudo systemctl restart rabbitmq-server  
// 这时候就可以访问 http://localhost:15672 使用默认用户名和密码（guest/guest）登录来管理交换机、队列等
```
---  
### 创建C++客户端demo使用rabbitmq
#### 安装C++的客户端代理库
```bash
# rabbitmq-c依赖
sudo dnf install librabbitmq-devel-0.13.0-4.tl4.x86_64
# SimpleAmqpClient
git clone https://github.com/alanxz/SimpleAmqpClient.git
cd SimpleAmqpClient
mkdir build
cd build
cmake ..
make
sudo make install
# 更新库
sudo ldconfig
```
#### 编写测试demo并运行
```c++
//client-send.cc
#include <SimpleAmqpClient/SimpleAmqpClient.h>
#include <iostream>

int main() {
    try {
        // 创建连接
        AmqpClient::Channel::ptr_t channel = AmqpClient::Channel::Create("localhost");

        // 声明队列
        std::string queue_name = "hello";
        channel->DeclareQueue(queue_name, false, true, false, false);

        // 创建生产者消息
        std::string message = "Hello World!";
        AmqpClient::BasicMessage::ptr_t msg = AmqpClient::BasicMessage::Create(message);
        std::cout << "=== produce message: " << message << std::endl;

        // 发送消息
        while (true) {
            channel->BasicPublish("", queue_name, msg);
            std::cout << "=== Sent " << message << std::endl;
            usleep(200 * 1000);
        }
        
    } catch (const std::exception &e) {
        std::cerr << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```
```c++
//client-receive.cc
#include <SimpleAmqpClient/SimpleAmqpClient.h>
#include <iostream>

int main() {
    try {
        // 创建连接
        AmqpClient::Channel::ptr_t channel = AmqpClient::Channel::Create("localhost");

        // 声明队列
        std::string queue_name = "hello";
        channel->DeclareQueue(queue_name, false, true, false, false);

        // 消费者标记
        std::string consumer_tag = channel->BasicConsume(queue_name, "");

        std::cout << " [*] Waiting for messages. To exit press CTRL+C" << std::endl;

        while (true) {
            AmqpClient::Envelope::ptr_t envelope = channel->BasicConsumeMessage(consumer_tag);
            std::string message_body = envelope->Message()->Body();
            std::cout << "=== Received " << message_body << std::endl;
            usleep(10 * 1000);
        }
    } catch (const std::exception &e) {
        std::cerr << "Error: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```
```bash
# 编译文件
g++ -o client-send client-send.cc -lSimpleAmqpClient -lrabbitmq
g++ -o client-receive client-receive.cc -lSimpleAmqpClient -lrabbitmq
export LD_LIBRARY_PATH=/usr/local/lib64:$LD_LIBRARY_PATH
./client-send  # 再运行程序
//另起一个终端
export LD_LIBRARY_PATH=/usr/local/lib64:$LD_LIBRARY_PATH
./client-receive  # 再运行程序
```
