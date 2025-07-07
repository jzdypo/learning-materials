# learning-materials  


## rabbitmq学习（C++，centos）  
### 在linux上安装rabbitmq服务  
#### 进行一些前置操作
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc'  
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key'    
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key'  
#### 安装rabbitmq-server并启动
dnf update -y  
dnf install -y logrotate  
dnf install -y erlang rabbitmq-server  
systemctl enable rabbitmq-server  
#### 安装管理插件并重启服务
sudo rabbitmq-plugins enable rabbitmq_management  
sudo systemctl restart rabbitmq-server  
// 这时候就可以访问 http://localhost:15672 使用默认用户名和密码（guest/guest）登录来管理交换机、队列等
---  
### 创建C++客户端demo使用rabbitmq

