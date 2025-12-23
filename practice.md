## 关卡一 ： 下载docker
### 1.打开终端：
```
# 进入你的个人主目录
cd ~ 
#创建一个专门放 Home Assistant 配置的文件夹
mkdir -p homeassistant/config
#进入这个文件夹查看路径
cd homeassistant/config
pwd
通常是 /home/ubuntu/homeassistant/config
```

### 安装docker引擎
#### 1. 更新索引并安装必要的工具
```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
```
#### 2. 添加 Docker 官方的密钥（保证下载安全）
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

上一步被拦截的话，就换国内镜像源添加密钥
```
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
#### 3. 把 Docker 的下载地址加入系统的“白名单”
```
这一行命令很长，建议直接复制粘贴：
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

##### 下载地址指向国内服务器
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### 4. 正式安装 Docker 引擎
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
如何确认是否成功？
安装完成后，再次输入：
docker --version
```



### 不知道这段代码还有没有用了，要是之前的操作都ok，这一步就可以跳过了
视频里是在绿联云的 Docker 镜像仓库搜“homeassistant”，在树莓派上，你直接把这行命令全部复制并粘贴到终端按回车：
```
sudo docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Asia/Shanghai \
  -v /home/ubuntu/homeassistant/config:/config \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
  ```


#### 好的下好docker，开始下一步
## 关卡二 ： 把 Home Assistant 的“集装箱（容器）”放到这个地基上。

### 1. 第一步：设置镜像加速（这一步非常重要！）
由于 Home Assistant 的镜像文件在国外，直接下载可能会卡住或报错。我们先给 Docker 加上“加速器”：

##### 1.打开配置文件：
```
sudo nano /etc/docker/daemon.json
```
##### 2.把下面这段话复制进去：
```
JSON

{
  "registry-mirrors": [
    "https://mirror.baidubce.com",
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com"
  ]
}
```
##### 3.保存并退出： 按 Ctrl + O，回车，再按 Ctrl + X。

##### 4.重启 Docker 使其生效：
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### 第二步：一键部署 Home Assistant
现在，我们执行最后那段长命令。这段代码对应视频里博主在界面上填写的“存储空间”、“网络模式”和“环境变量”。

###### 直接复制这一整段到终端：
```
sudo docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Asia/Shanghai \
  -v /home/$USER/hass_config:/config \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
```
这一步在做什么？（对应视频里的设置）
```
--name homeassistant：给你的容器起名。

-v /home/$USER/hass_config:/config：对应视频里的存储空间。它会在你的主目录下创建一个 hass_config 文件夹，所有的配置都在这。

--network=host：对应视频里的网络模式：Host。这样它才能找到你家里的智能灯。

ghcr.io/...:stable：这就是你要下载的镜像（Image）。
```
### 第三步：等待并进入网页
1.运行完上面的命令后，终端会显示一串长长的 ID。

2.输入 sudo docker ps 确认一下，如果看到 homeassistant 状态是 Up，说明它已经在运行了！

3.打开浏览器（用你的电脑或者树莓派里的浏览器），在地址栏输入： http://localhost:8123 （如果在树莓派本地） 或者 http://树莓派的IP地址:8123 （在另一台电脑上）