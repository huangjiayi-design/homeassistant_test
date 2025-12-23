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
