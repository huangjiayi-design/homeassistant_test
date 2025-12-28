# Raspberry Pi Home Assistant 落地实战手册 (手机热点版)

## 🛠 环境准备
- 硬件：Raspberry Pi 4B/5（ARM64 架构）
- 系统：Ubuntu Desktop / Server 22.04+
- 网络：建议准备两台移动设备（一台开热点，一台操作），或使用路由器。
## 🚀 快速开始
### （一） 基础环境搭建 (Docker)

#### 1.打开终端：
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

#### 2. 更新索引并安装必要的工具
```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
```
#### 3.添加 docker 官方密钥

```
sudo mkdir -p /etc/apt/keyrings
```
使用国内镜像源添加密钥：
```
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

#### 4. 把 Docker 的下载地址加入系统的“白名单”，写入软件源

```

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### 5. 正式安装 Docker 引擎
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
如何确认是否成功？
安装完成后，再次输入：
docker --version
```

### （二） Home Assistant 部署

> 针对 ARM64 架构，建议强制指定平台镜像以确保兼容性。

因为Home Assistant的镜像源在国外，直接下载可能会卡住，所以我们使用国内的镜像源下载：
#### 1. 拉取镜像源
```
sudo docker pull docker.m.daocloud.io/homeassistant/home-assistant:stable
```
#### 2. 检查镜像详细信息（关键：确认架构）
```
sudo docker inspect docker.m.daocloud.io/homeassistant/home-assistant:stable | grep Architecture
```
#### 3. 强制以“指定架构”模式运行
在运行命令时，我们再次声明平台，并使用刚才下载的新地址：
- **注意**：此处我们将配置路径统一为 ```~/hass_config```
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

#### 4.确认运行状态：
>>```
>>sudo docker ps
>>```
>> * 找到 homeassistant 那一行。
>> * 检查 STATUS 这一列。如果是 Up ... seconds 或者 Up ... minutes，那就彻底稳了！
>> * 如果它显示 Restarting...，说明还有小问题，但通常现在应该是 Up。
#### 5. 进入Home Assistant界面
>> * 现在可以离开终端，回到你的电脑（Windows/Mac）浏览器了
>> * 打开浏览器（建议用 Chrome、Edge 或 Safari）。
>> * 输入地址： http://你的树莓派IP:8123
>>（如果你忘了 IP，在树莓派输入 ```hostname -I``` 查看）
>> * 等待： 第一次启动可能需要 1-2 分钟来初始化数据库。

>> ## 因为电脑和树莓派连接的都是手机的热点：
>> 所以在电脑上http:树莓派IP:8123就打不开这个网页
![alt text](image.png)
情况如上

> 树莓派上：通过查看HomeAssistant实时日志：
>>```
>>sudo docker logs --tail 20 homeassistant
>>```
>- 正常情况：你应该看到类似 Home Assistant is running 或者 Starting Home Assistant...。
>- 异常情况：如果没有任何输出，或者全是 Error，说明容器内部出问题了。
> ![alt text](e40aa5c81e82d1461c0b01e532451c3b_720.jpg)

>> 但是还好，根据树莓派上的Error:日志里全是 habluetooth.scanner 和 hci0。这只是说 Home Assistant 在尝试搜索你家里的蓝牙设备，但树莓派的蓝牙没配置好或者被占用了。这完全不影响你进入网页系统。
>> 关键的问题在于：
中间有一行 Network unreachable，是因为用的是手机热点，Home Assistant 发现没法连接到互联网去下载天气、时间等插件，这也不影响进入本地后台。
>### Plan A:直接使用树莓派自己的浏览器打开
>>地址栏直接输入：http://localhost:8123 或者 http://127.0.0.1:8123
如果能打开，安装就是没问题的
> ### Plan B: 解决手机热点屏蔽
>>* 在树莓派的终端输入： sudo ufw allow8123/tcp (手动放行端口)
>>* 检查电脑连接的是同一个手机热点
如果还是不行：用 USB 线把手机连到电脑，开启 “USB 共享网络”，这样电脑和手机热点就在同一个更直接的局域网里了。


### （三）  插件商店 (HACS) 手动安装
- **难点**：热点环境下 GitHub 访问不稳定，建议手动下载 zip 包。
####  1.重新新建一个文件夹与 homeassistant文件夹 同级
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

这是因为Docker创建的```hass_config```默认为root权限，需手动创建并修正权限。

>> 但是没关系：
>##### 第一步 ：借用‘超级力量’ 创建
>>在命令前面加上 sudo，这表示“以管理员身份执行”：
>>```
>>sudo mkdir -p /home/$USER/hass_config/custom_components
>>sudo mkdir -p /home/$USER/hass_config/www

> ##### 第二步：把文件夹的“所有权”拿回来
>> 即便创建了文件夹，如果“主人”还是 root，Home Assistant 以后可能没法往里写数据。我们需要把整个配置文件夹的权限交还给你现在的用户（假设你的用户名是 ubuntu）：
>>```
>>#将文件夹的所有者改为当前用户
>>sudo chown -R $USER:$USER /home/$USER/hass_config/
>>
>>
>>#给文件夹赋予读写权限
>>sudo chmod -R 755 /home/$USER/hass_config/
>>
>**验证一下**：输入ls -l  如果文件夹旁边的不是 root root,而是 ubuntu ubuntu,说明权限修正成功

#### 2.下载hacs文件
>在此之前记得要先在custom_components文件夹中创建一个hacs文件夹
https://github.com/hacs/integration/releases/latest/download/hacs.zip
>> * 好的下载hacs文件之后进行解压，将解压缩之后的文件都放到custom_components文件夹中的hacs文件夹。
>#### 再次修正权限：
>>因为你手动拷入了新文件，为了保险，再次运行一次权限指令，确保 Home Assistant 能读到它们：
>>```
>>sudo chown -R $USER:$USER /home/$USER/hass_config/custom_components/hacs/
>>sudo chmod -R 755 /home/$USER/hass_config/custom_components/hacs/
>>

#### 3. 重启homeassistant


>1. 回到 Home Assistant 网页.
> 2. 点击 配置 (Settings) -> 系统 (System) -> 右上角 重新启动 (Restart)。
> 3. 或者选择 重新启动 Home Assistant。会有一个中断正在运行的自动化和脚本按钮，是的没错点击重启homeassistant

#### 4. 在界面中"添加集成"
>>重启完成后（可能需要 1-2 分钟）：
>1. 点击 **配置 (Settings)** -> **设备与服务 (Devices & Services)**。
>2. 点击右下角的 **添加集成 (Add Integration)。**
>3. **搜索 HACS.**
>>注意： 如果搜不到，按 Ctrl + F5 强制刷新网页缓存后再搜。
>4. 之后点击 **HACS**这一栏，会弹出一个框，会出现4或5个框框，然后勾选上，点击**提交**
>5. 之后弹出一个新的小框，有一行**github的链接**和下面**一行验证码**，把验证码给复制下来，点击上面那一行链接,之后这里如果没有github账号的话，需要自己再创建一个，创建好账号之后，再把验证码粘贴到下面的框框中，点击**Continue**，然后再点击**Authorize hacs**
>>之后回到homeassistant,我现在的页面是这样的：
>>![alt text](0894e4f5f9d10cd92f7e0c5045e6423a_720.jpg)
>> 这里可以选择直接跳过，也可以选“客厅”、“办公室”

>Home Assistant 的左边边栏多了一个图标写着“HACS”，点击进入 HACS 后，你可能会看到它在“转圈”或者提示“正在背景下载数据”。这是因为它正在从 GitHub 拉取成千上万个插件的目录列表。


### （四）  智能家居设备接入
#### 小米 (Xiaomi MIoT)：支持账号集成。
> 1. 在左边的边栏点击HACS，在这里直接搜索“xiaomi miot"
> 2. 直接点击，然后download，之后弹出来一个小框，依然是点击download，等待部署完成，
> 3. 部署完成之后，在设置里面点击最上面的 **一个修复** 那里点击，弹出一个"Restart required"的警告，点击“提交”->重启Homeassistant,等待几分钟。
> 4. 点击设置， 点击**设备与服务**， 点击**添加集成**， 搜索**xiaomi miot**，点击添加，这里会弹出一个框来，**选择操作**：这里我们选择“**Add divices using Mi Account**" (账号集成) **->** 点击**下一步**

#### 美的 (Midea AC LAN)：重点讲解如何通过热点管理界面寻找空调 IP。
>1. 首先在HACS里卖搜索Midea AC LAN ->download->download->重启HomeAssistant
>2. 在添加集成之前，需要在手机上安装一个美的美居，然后在这个美的美居上创建好**账号**，设定好**密码**，添加设备，这里以空调为例，手机如果是连热点的话建议再找一台设备A用来开热点，设备B连设备A的热点，之后用设备B与空调连接的时候，会弹出一个空调的WIFI，然后用设备B去连接，连接好了之后就可以用设备A在热点里面找空调的**IP**，
>3. 设置里面重启之后，点击**设备与服务**， 点击**添加集成**， 搜索**Midea AC LAN**，点击添加，这里会弹出一个框来，**选择操作**：这里我们选择“**Discover automatically**" (自动发现) , 删掉输入栏中的“auto”，然后输入**空调IP地址**，然后就点下一步和选择是哪个位置的设备，一般都已经写好了。
>>之后就可以在概览那里看到新添加的空调控制窗口了。


如果你使用的是 docker run 命令行
你需要停止并删除旧容器，用新的命令重新运行（数据不会丢，因为你映射了 hass_config）。

你的新命令需要包含 --device /dev/gpiomem 和 --privileged：
```
docker stop homeassistant
docker rm homeassistant

docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Asia/Shanghai \
  -v /home/$USER/hass_config:/config \
  --network=host \
  --device /dev/gpiomem:/dev/gpiomem \
  --device /dev/mem:/dev/mem \
  docker.m.daocloud.io/homeassistant/home-assistant:stable
```


##### 修改后的正确格式
```yaml
switch:
  - platform: rpi_gpio
    switches:
      - port: 17
        name: "Desk LED"
        unique_id: "raspberry_pi_led_17"
```

PWM 软件设计：

```yaml
# 1. 首先确保你的基础开关配置正确 (注意 entity_id 会是 switch.desk_led)
switch:
  - platform: rpi_gpio
    switches:
      - port: 17
        name: "Desk LED"

# 2. 定义一个虚拟的亮度数值存储 (用于滑块滑动)
input_number:
  led_brightness:
    name: "LED Brightness Helper"
    initial: 255
    min: 0
    max: 255
    step: 1

# 3. 定义模板灯
template:
  - light:
      - name: "My LED with Slider"
        unique_id: "my_dimmable_led_new"
        # 状态显示：根据物理开关判断是开还是关
        state: "{{ is_state('switch.desk_led', 'on') }}"
        # 亮度显示：从辅助器读取数值
        level: "{{ states('input_number.led_brightness') | int }}"
        
        # 点开灯时的动作
        turn_on:
          action: switch.turn_on
          target:
            entity_id: switch.desk_led
            
        # 点关灯时的动作
        turn_off:
          action: switch.turn_off
          target:
            entity_id: switch.desk_led
            
        # 滑动滑块时的动作
        set_level:
          # 保存滑动后的数值
          - action: input_number.set_value
            target:
              entity_id: input_number.led_brightness
            data:
              value: "{{ brightness }}"
          # 根据亮度决定是否开启物理开关
          - action: >
              {% if brightness > 0 %}
                switch.turn_on
              {% else %}
                switch.turn_off
              {% endif %}
            target:
              entity_id: switch.desk_led
```

```yaml
switch:
  - platform: rpi_gpio
    switches:
      - port: 17             # 这是你之前的 LED
        name: "Desk LED"
        unique_id: "raspberry_pi_led_17"
      - port: 18             # 直接并列在下面，不要重复写 switch:
        name: "Fan Switch"
        unique_id: "pi_fan_ctrl"
```

包装：
```yaml
# 1. 基础开关 (物理层)
switch:
  - platform: rpi_gpio
    switches:
      - port: 17
        name: "Desk LED Switch"
      - port: 18
        name: "Fan Switch"

# 2. 统一模板域 (界面层)
# 注意：整个 yaml 文件只能有一个顶层 template: 标签
template:
  # 这里放你之前的灯
  - light:
      - name: "My LED with Slider"
        unique_id: "my_dimmable_led_new"
        state: "{{ is_state('switch.desk_led_switch', 'on') }}"
        level: "{{ states('input_number.led_brightness') | int }}"
        turn_on:
          action: switch.turn_on
          target:
            entity_id: switch.desk_led_switch
        turn_off:
          action: switch.turn_off
          target:
            entity_id: switch.desk_led_switch
        set_level:
          - action: input_number.set_value
            target:
              entity_id: input_number.led_brightness
            data:
              value: "{{ brightness }}"

  # 这里放你的新版风扇
  - fan:
      - name: "我的小风扇"
        unique_id: "my_desk_fan_new"
        state: "{{ states('switch.fan_switch') }}"
        turn_on:
          action: switch.turn_on
          target:
            entity_id: switch.fan_switch
        turn_off:
          action: switch.turn_off
          target:
            entity_id: switch.fan_switch
```


### PWM硬件方面的实现：
如果你是用 Docker 容器手动安装的 Home Assistant Core，你是看不到“加载项”菜单的。如果是这种情况，你需要手动在树莓派命令行安装 Mosquitto：sudo apt install mosquitto mosquitto-clients -y。

既然你是在树莓派的 Docker 宿主机上直接通过 apt 安装了 Mosquitto，那么你的 MQTT 服务现在是直接运行在 Linux 系统层（宿主机）上的，而不是作为一个 Docker 容器。

##### 1.启动并检查 Mosquitto 服务
在终端执行以下命令，确保 MQTT 服务已经跑起来了：
```
# 启动服务
sudo systemctl start mosquitto
# 设置开机自启
sudo systemctl enable mosquitto
# 检查状态
sudo systemctl status mosquitto
```

如果你看到 active (running)，说明“邮局”（Broker）已经开张了。

##### 2.获取宿主机的 IP 地址
因为你的 Home Assistant 跑在 Docker 里，它需要知道宿主机的 IP 才能连接到这个 MQTT 服务。 在终端输入：
```
hostname -I
```
通常会得到类似 192.168.x.x 的地址。记下这个 IP。
10.211.181.9. 172.17.0.1

10.211.181.9：这是你的树莓派在局域网（或是虚拟机网络）中的真实身份。
172.17.0.1：这是 Docker 网桥 (Docker Bridge) 的地址。对于运行在 Docker 容器里的 Home Assistant 来说，这个地址就是通往“宿主机物理世界”的门户。


##### 3.在 Home Assistant 中集成 MQTT
现在你需要告诉 HA 去连接这个刚刚装好的 Mosquitto。

1. 打开 HA 界面，点击 设置 (Settings) -> 设备与服务 (Devices & Services)。
2. 点击右下角的 添加集成 (Add Integration)。
3. 搜索并选择 MQTT。
4. 在配置界面输入：
- 代理 (Broker): 输入你刚才查到的宿主机 IP（如果是 Host 模式也可以试着填localhost）。
- 端口 (Port): 1883。
- 用户名/密码: 如果你刚才安装后没设，就先留空。
5. 点击 提交。

##### 但是我在提交的时候显示：连接失败：
>
>###### 1. 这个时候就需要打开文件mosquitto.conf文件了,可能是文件配置的问题
>```
>sudo nano /etc/mosquitto/mosquitto.conf
>```
>打开之后，检查文件，确保有以下内容,没有的话添加到文件最后面
>```
>listener 1883 0.0.0.0
>allow_anonymous true
>```
>
>0.0.0.0：强制 Mosquitto 监听所有网卡接口（包括 Docker 网桥接口）。
>allow_anonymous true：允许匿名连接（如果你还没设置用户名密码）。
>
>保存退出（Ctrl+O, Enter, Ctrl+X）。
>**之后重启mosquitto服务**：
>```
>sudo systemctl restart mosquitto
>```
>###### 2. 查看防火墙
>如果 Mosquitto 配置正确但仍连接失败，可能是树莓派的防火墙（ufw）拦截了 1883 端口。
>输入以下命令查看防火墙状态：
>```
>sudo ufw status
>```
>如果是inactive的话就没什么问题
>如果是**active**, 请放行 1883 端口：
>```
>sudo ufw allow 1883
>```
>我这里查看防火墙的时候是inactive，所以就是前面mosquitto.conf文件的配置问题
>###### 在用homeassistant连接之前测试一下：
>重新开一个终端页面，输入以下指令，回车，如果在终端页面显示 hello ,即连接成功
>```
>mosquitto_pub -h 172.17.0.1 -t "test/topic" -m "Hello"
>```
>- 如果成功：没有任何报错，说明宿主机服务已准备好接受来自 Docker 网桥的连接。
>- 如果失败：说明 Mosquitto 依然没配置好或者正在拒绝非 127.0.0.1 的 IP。
>
>###### 备选方案：
>备选 IP 方案
>如果 172.17.0.1 始终连接失败，你可以尝试在 Home Assistant 的配置界面使用你的局域网物理 IP：>10.211.181.9   
>这个我试了一下，也是可以连接的

##### 4.写python脚本并运行：
HA 发送指令到 MQTT 代理，Python 脚本读取指令并控制 GPIO。
###### (1). 安装 Python 环境和库： 在树莓派终端执行：
```
sudo apt update
sudo apt install python3-pip python3-rpi.gpio -y
pip3 install paho-mqtt
```
###### (2). 编写 Python 控制脚本
在树莓派上创建一个名为 fan_controller.py 的文件，并写入以下内容：
同时这个fan_controller.py文件的放置位置：
```py
import RPi.GPIO as GPIO
import paho.mqtt.client as mqtt

# --- 配置区 ---
FAN_PIN = 18
MQTT_BROKER = "localhost"
MQTT_TOPIC = "home/fan/set_speed"      #和 HA 发布的主题一样，

# --- GPIO 初始化 ---
GPIO.setmode(GPIO.BCM)
GPIO.setup(FAN_PIN, GPIO.OUT)
# 设置频率为 100Hz
pwm = GPIO.PWM(FAN_PIN, 100)           #频率 100Hz，每秒开关100次
pwm.start(0)                           #占空比 0~100，起初占空比为0

# --- MQTT 回调函数 ---
def on_message(client, userdata, message):   
    try:
        # 接收 0-100 的数值
        speed = int(message.payload.decode("utf-8"))  #读取payload , 如 b'50'
        print(f"设置风扇速度为: {speed}%") 
        pwm.ChangeDutyCycle(speed)                    # 实际控制树莓派 GPIO 输出 PWM 信号,pwm是一个对象，可以调用它的方法，比如.ChangeDutyCyctle()。
    except ValueError:
        pass

# --- 启动 MQTT 监听 ---
client = mqtt.Client()        #创建一个新的 MQTT 客户端实例。mqtt.CallbackAPIVersion.VERSION2：指定使用新版回调接口（避免警告，更稳定）。
client.on_message = on_message #告诉客户端：“以后一有消息来，就调用我写的 on_message函数！”
client.connect(MQTT_BROKER, 1883, 60) #连接到 MQTT 服务器（Broker）。
#参数：
#地址：localhost（本机）
#端口：1883（MQTT 默认端口）
#超时时间：60秒
client.subscribe(MQTT_TOPIC)         #订阅消息
print("风扇中继脚本已启动，等待指令...")
client.loop_forever()                #进入无限循环，持续监听网络消息。
```

后面一段脚本的修改
```py
# --- 启动 MQTT 监听 ---
# 指定 Callback API 版本以兼容最新库
client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2) 
client.on_message = on_message
client.connect(MQTT_BROKER, 1883, 60)
client.subscribe(MQTT_TOPIC)
print(f"风扇中继脚本已启动！监听主题: {MQTT_TOPIC}")
client.loop_forever()
```

| 位置  | 角色 | 填写的IP地址 | 理由 |
| :----: | :----: | :----: | :----: |
|HA MQTT 集成界面 | 发信人 | 172.17.0.1 | 从容器跳向宿主机 |
|Python 脚本变量 | 收信执行人 | 127.0.0.1 | 在宿主机内部通信 |
| Mosquitto 配置文件 | 邮局/中转 | 0.0.0.0 | 允许监听来自所有方向（包括 HA）的连接 |

##### 5. 在 Home Assistant 中接入
组件一：  在 configuration.yaml 中，你可以定义一个 MQTT 平台的速度辅助器：
```yaml
# 1. 定义一个滑动条
input_number:
  fan_speed_slider:
    name: "风扇调速"
    initial: 0
    min: 0
    max: 100
    step: 1


# 2. 自动化：滑动条变动时发 MQTT 消息
automation:
  - alias: "Sync Fan Speed to MQTT"
    trigger:
      platform: state
      entity_id: input_number.fan_speed_slider
    action:
      action: mqtt.publish
      data:
        topic: "home/fan/set_speed"
        payload: "{{ states('input_number.fan_speed_slider') | int }}"
```

组件二：
我们需要一个 UI 组件来发送 MQTT 消息。最快的方法是修改 configuration.yaml：
打开 configuration.yaml，添加一个滑块和对应的自动化：
```yaml
# 1. 创建一个 0-100 的数值滑块
input_number:
  fan_speed_control:
    name: "风扇转速百分比"
    initial: 0
    min: 0
    max: 100
    step: 1
    unit_of_measurement: "%"
    icon: mdi:fan

# 2. 自动化：当滑块变动时，向 MQTT 发送消息
automation:
  - alias: "推送风扇速度到MQTT"
    trigger:
      platform: state
      entity_id: input_number.fan_speed_control
    action:
      - action: mqtt.publish
        data:
          topic: "home/fan/set_speed"
          payload: "{{ states('input_number.fan_speed_control') | int }}"
```

注释版本：
```yaml
input_number:
  fan_speed_control:       # 系统的“身份证号”(Entity ID)，不可重复
    name: "风扇转速百分比"  # 界面显示的友好名称
    initial: 0             # 每次重启 HA 后滑块默认的位置
    min: 0                 # 最小值
    max: 100               # 最大值
    step: 1                # 每次滑动的最小间隔
    unit_of_measurement: "%" # 在数字后面显示的单位
    icon: mdi:fan          # 组件旁边的图标（一个风扇图标）
```
它就像一个中转站。你在屏幕上拖动滑块，实质上是在改变这个变量的值。但仅仅改变这个值，风扇是不会转的，因为它还没和 MQTT 关联起来。

automation：定义“联动规则”
这部分是“大脑”，负责把滑块的变化翻译成MQTT指令发出去:
```yaml
automation:
  - alias: "推送风扇速度到MQTT" # 自动化的名字
    trigger:                   # 【触发器】：什么时候开始工作？
      platform: state          # 监控状态变化
      entity_id: input_number.fan_speed_control # 只要滑块被拖动了
    action:                    # 【动作】：要做什么？
      - action: mqtt.publish   # 执行发送 MQTT 消息的动作
        data:
          topic: "home/fan/set_speed" # 消息的目的地（必须和 Python 脚本里的一致）
          # payload 是具体的信件内容：
          # {{ ... }} 是一段模板代码，意思是：取出滑块当前的状态值，并转为整数(int)
          payload: "{{ states('input_number.fan_speed_control') | int }}"
```
含义：一旦触发器发现滑块变了（比如从 20 变到了 50），它就会抓取这个“50”，打包成 MQTT 报文发给宿主机的 Mosquitto 邮局。



```yaml
input_number:  #定义一个输入实体
  fan_speed_slider:  #创建一个fan_speed_slider的输入数字实体，在 Home Assistant 前端会显示为一个滑块或输入框，用户可以通过它设置风扇速度。
    name: "风扇调速"   #显示在界面上的名称，中文"风扇调速"
    initial: 0        #初始值为 0
    min: 0            #最小值为 0
    max: 100          #最大值为 100
    step: 1           #调节的步长 为 1


# 2. 自动化：滑动条变动时发 MQTT 消息
#当fan_speed_slider 的值发生变化时，自动通过 MQTT协议发布一条消息，把当前速度值发送给支的MQTT的风扇设备，这时就使用到我们写的脚本了
automation:                #
  - alias: "Sync Fan Speed to MQTT"
    trigger:               #触发器
      platform: state      #监听 input_number.fan_speed_slider这个实体的状态变化，只要用户拖动滑块或修改这个数值，就触发这个自动化
      entity_id: input_number.fan_speed_slider
    action:                #动作，执行MQTT发布动作
      action: mqtt.publish #执行MQTT发布动作
      data:
        topic: "home/fan/set_speed"   #消息发送到MQTT的主题，通常风扇设备会订阅这个主题，收到消息后调整风速
        payload: "{{ states('input_number.fan_speed_slider') | int }}"  #把输入实体的数据传过来，同时，将其传过来的字符串形式转换成整型数字
```