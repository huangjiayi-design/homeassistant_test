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
```但是有点问题 在安装上
在此之前：安装curl：sudo apt update && sudo apt install curl -y

curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

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
#### 3. 把 Docker 的下载地址加入系统的“白名单”
```
这一行命令很长，建议直接复制粘贴：
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
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

### 
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
