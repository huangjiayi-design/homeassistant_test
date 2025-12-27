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



>>##### 1. 进到你放插件的目录
>>cd ~/hass_config/custom_components/
>>##### 2. 从网上下载这个插件代码
>>git clone https://github.com/thecode/ha-rpi_gpio.git rpi_gpio_temp
>>##### 3. 把核心文件夹搬到正确的位置
>>mv rpi_gpio_temp/custom_components/rpi_gpio ./rpi_gpio
>>##### 4. 把用不着的垃圾清理掉
>>rm -rf rpi_gpio_temp
>>
>>##### 5. 按照你的 readme 关卡三，把所有权拿回来（这步不做，HA 就看不见插件！）
>>
>>```
>>sudo chown -R $USER:$USER ~/hass_config/
>>sudo chmod -R 755 ~/hass_config/
>>```
>>```nano ~/hass_config/configuration.yaml```
>>粘贴进去这些代码：
```
switch:
  - platform: rpi_gpio
    switches:
      - port: 18
        name: "School_Project_LED"
        unique_id: "school_led_001"
```

你的 README.md 避坑指南更新
现象：日志报错 'ports' is an invalid option。

原因：YAML 配置参数与插件版本不匹配，或缩进层级错误。

解决：查阅插件文档，将 ports: 结构改为 switches: 列表结构。

###### Docker 硬件权限:
如果在网页上显示 on 但灯不亮，可能是 Docker 没拿到操作 GPIO 的最终权限。这时可以在终端输入这个“大力出奇迹”的命令并重启：
```
sudo chmod 666 /dev/gpiomem
sudo docker restart homeassistant
```


###### 强制赋予 Docker 硬件访问权限
>>```
>>#彻底放开 GPIO 内存访问权限
>>sudo chmod 777 /dev/gpiomem
>>#确保 ubuntu 用户在 gpio 组（如果报错说明已存在，没关系）
>>sudo usermod -a -G gpio $USER
>#重启 HA 容器让权限生效
>sudo docker restart homeassistant
>```
##### 这里我的led灯还是没亮：
sudo docker inspect homeassistant | grep -i privileged
-> "Privileged":true
这说明 Docker 容器与树莓派硬件之间的“大门”是完全敞开的。

我们直接在树莓派终端，绕过 Home Assistant，用系统自带的命令控制引脚。如果这招亮了，就说明是 HA 插件的兼容性问题；如果这招都不亮，就是引脚接触或数错位置的问题。
```
# 1. 进入 GPIO 控制目录
cd /sys/class/gpio

# 2. 导出 GPIO 17（如果你配置改成了 17）
# 如果报错说 device or resource busy，说明 HA 正在占用它，不用管，直接跳到第 4 步
echo 17 | sudo tee export

# 3. 设置为输出模式
echo out | sudo tee gpio17/direction

# 4. 强制给高电平（点火！）
echo 1 | sudo tee gpio17/value
```


> 这里在执行第二步的时候：
>>```
>>echo 17 | sudo tee export
>>输出结果：
>>17 
>>tee: export: Invalid argument
>>```
>它通常意味着 GPIO 17 已经被系统“占用”或“导出”过了，或者系统正在使用不同的编号规则。

>好的这里我们直接跳到第四步：
>>```
>>echo 1 | sudo tee gpio17/value
>>输出结果：
>>tee: gpio17/value: No such file or directory
>>1
>>```
> - No such file or directory 终于让我们找到了最后的“病灶”：GPIO 17 根本没有被成功激活（导出）。
> - 这是因为在树莓派 4B 的较新系统中，传统的 /sys/class/gpio 操作方式正在被废弃，或者由于之前的 Invalid argument 报错，导致 gpio17 这个文件夹压根没生成。


#### 终极方案：使用“远程 GPIO”插件（万能灵药）

现在的 rpi_gpio 插件在很多 Docker 环境下确实存在映射不上的问题。我们换一个更强大、兼容性更好的办法，也是老师会非常欣赏的高级操作。

#### 第一步：在树莓派宿主机安装 GPIO 守护进程
回到你的树莓派终端（不要进入容器），输入：
```
#安装 GPIO 守护服务
sudo apt update
sudo apt install pigpio -y

#启动并设置为开机自启
sudo systemctl enable pigpio
sudo systemctl start pigpio
```

#### 第二步：修改 Home Assistant 配置
我们要改用 remote_rpi_gpio 模式，它通过网络端口通信，完美绕过 Docker 的权限和路径问题。

输入 ```nano ~/hass_config/configuration.yaml```。

把原来的 ```switch``` 部分全部删掉，改为：
```
switch:
  - platform: remote_rpi_gpio
    host: 127.0.0.1
    ports:
      17: "School_Project_LED"
```
保存退出并重启HA：```sudo docker reatart homeassistant```


可是在这里的：```sudo apt install pigpio -y``` 的输出结果是：```E: Package 'pigpio' has no installation candidate ```

##### 使用系统原生的 gpiod 工具进行控制
```
switch:
  - platform: command_line
    switches:
      school_project_led:
        # 注意：在树莓派4B上，GPIO 17 通常对应芯片 0 的第 17 号偏移
        command_on: "gpioset 0 17=1"
        command_off: "gpioset 0 17=0"
        friendly_name: "School_Project_LED"
```