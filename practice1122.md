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

## 关卡二： 安装docker引擎

#### 1. 更新索引并安装必要的工具
```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
```
#### 2.添加 docker 官方密钥

```
sudo mkdir -p /etc/apt/keyrings
```
使用国内镜像源添加密钥：
```
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

#### 3. 把 Docker 的下载地址加入系统的“白名单”

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

## 关卡三 ： 把 Home Assistant 的“集装箱（容器）”放到这个地基上。  安装镜像以及启动homeassistant

因为Home Assistant的镜像源在国外，直接下载可能会卡住，所以我们使用国内的镜像源下载：
1. 拉取（注意：这次换了地址，成功率极高）
```
sudo docker pull docker.m.daocloud.io/homeassistant/home-assistant:stable
```
2. 检查镜像详细信息（关键：确认架构）
```
sudo docker inspect docker.m.daocloud.io/homeassistant/home-assistant:stable | grep Architecture
```
第三步：强制以“指定架构”模式运行
在运行命令时，我们再次声明平台，并使用刚才下载的新地址：
```
sudo docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  --platform linux/arm64 \
  -e TZ=Asia/Shanghai \
  -v /home/$USER/hass_config:/config \
  --network=host \
  docker.m.daocloud.io/homeassistant/home-assistant:stable

```

>>### 确认运行状态：
>>```
>>sudo docker ps
>>```
>> * 找到 homeassistant 那一行。
>> * 检查 STATUS 这一列。如果是 Up ... seconds 或者 Up ... minutes，那就彻底稳了！
>> * 如果它显示 Restarting...，说明还有小问题，但通常现在应该是 Up。
>>### 进入Home Assistant界面
>> * 现在可以离开终端，回到你的电脑（Windows/Mac）浏览器了
>> * 打开浏览器（建议用 Chrome、Edge 或 Safari）。
>> * 输入地址： http://你的树莓派IP:8123
>>（如果你忘了 IP，在树莓派输入 hostname -I 查看）
>> * 等待： 第一次启动可能需要 1-2 分钟来初始化数据库。

>> ## 因为电脑和树莓派连接的都是手机的热点：
>> 所以在电脑上http:树莓派IP:8123就打不开这个网页
![alt text](image.png)
情况如上

>> 树莓派上：
>> ![alt text](e40aa5c81e82d1461c0b01e532451c3b_720.jpg)

>> 但是还好，根据树莓派上的Error:日志里全是 habluetooth.scanner 和 hci0。这只是说 Home Assistant 在尝试搜索你家里的蓝牙设备，但树莓派的蓝牙没配置好或者被占用了。这完全不影响你进入网页系统。
>> 关键的问题在于：
中间有一行 Network unreachable，是因为用的是手机热点，Home Assistant 发现没法连接到互联网去下载天气、时间等插件，这也不影响进入本地后台。
>>### Plan A:直接使用树莓派自己的浏览器打开
>>地址栏直接输入：http://localhost:8123 或者 http://127.0.0.1:8123
如果能打开，安装就是没问题的
>> ### Plan B: 解决手机热点屏蔽
>>* 在树莓派的终端输入： sudo ufw allow8123/tcp (手动放行端口)
>>* 检查电脑连接的是同一个手机热点
如果还是不行：用 USB 线把手机连到电脑，开启 “USB 共享网络”，这样电脑和手机热点就在同一个更直接的局域网里了。

## 关卡四：配置homeassistant
### 1.重新新建一个文件夹与 homeassistant文件夹 同级
```
#进入配置文件夹
cd ~/hass_config

# 创建 custom_components 文件夹
mkdir -p custom_components

# 创建 www文件夹
mkdir -p www
```

>> 有一个问题：
>>就是我在终端创建文件夹的时候说我没有权限:
>>```
>>mkdir: cannot create directory 'custom_components': Pession denied

>> 但是没关系：
>>##### 第一步 ：借用‘超级力量’ 创建
>>在命令前面加上 sudo，这表示“以管理员身份执行”：
>>```
>>sudo mkdir -p /home/$USER/hass_config/custom_components
>>sudo mkdir -p /home/$USER/hass_config/www
>>```

>> ##### 第二步：把文件夹的“所有权”拿回来
>> 即便创建了文件夹，如果“主人”还是 root，Home Assistant 以后可能没法往里写数据。我们需要把整个配置文件夹的权限交还给你现在的用户（假设你的用户名是 ubuntu）：
>>```
>>#将文件夹的所有者改为当前用户
>>sudo chown -R $USER:$USER /home/$USER/hass_config/
>>
>>#给文件夹赋予读写权限
>>sudo chmod -R 755 /home/$USER/hass_config/
>>```
>>验证一下：输入ls -l  如果文件夹旁边的不是 root root,而是 ubuntu ubuntu,说明权限修正成功

### 2.下载hacs文件
>>在此之前记得要先在custom_components文件夹中创建一个hacs文件夹
https://github.com/hacs/integration/releases/latest/download/hacs.zip
>> * 好的下载hacs文件之后进行解压，将解压缩之后的文件都放到custom_components文件夹中的hacs文件夹。
>>#### 再次修正权限：
>>因为你手动拷入了新文件，为了保险，再次运行一次权限指令，确保 Home Assistant 能读到它们：
>>```
>>sudo chown -R $USER:$USER /home/$USER/hass_config/custom_components/hacs/
>>sudo chmod -R 755 /home/$USER/hass_config/custom_components/hacs/
>>

### 3. 重启homeassistant
>>
>> 1. 回到 Home Assistant 网页.
>> 2. 点击 配置 (Settings) -> 系统 (System) -> 右上角 重新启动 (Restart)。
>> 3. 选择 重新启动 Home Assistant。

### 4. 在界面中"添加集成"
>>重启完成后（可能需要 1-2 分钟）：
>>1. 点击 **配置 (Settings)** -> **设备与服务 (Devices & Services)**。
>>2. 点击右下角的 **添加集成 (Add Integration)。**
>>3. **搜索 HACS.**
>>注意： 如果搜不到，按 Ctrl + F5 强制刷新网页缓存后再搜。

### 5.