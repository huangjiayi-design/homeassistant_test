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

##### 上面的还是不要看了，直接用下面的吧，下载地址指向国内服务器
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
```
### 1. 第一步：设置镜像加速（这一步非常重要！）
由于 Home Assistant 的镜像文件在国外，直接下载可能会卡住或报错。我们先给 Docker 加上“加速器”：

镜像加速的话老是有问题：Job for docker.service failed because the control process exited with error code. See "system status docker.service" and "journalctl -xeu docker.service " for details. （一到重启就报这样的错)
所以就省掉这一步了
##### 1.重启 Docker 使其生效：
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

如果上面在拉取镜像时很慢，试着先用国内的镜像源拉取：
###### 第一步：改用网易或百度维护的镜像地址（速度通常是官方的 10 倍以上）。
```
sudo docker pull hub-mirror.c.163.com/homeassistant/home-assistant:stable
```
sudo docker pull hub-mirror.c.163.com/homeassistant/home-assistant:stable 在这一步下：Error response from deamon:failed to resolve reference :"hub-mirror.c.163.com/homeassistant/home-assistant:stable" :failed to do request : Head "https://hub-mirror.c.163.com/homeassistant/home-assistant:stable" : dial tcp:lookup hub-mirror.c.163.com on 127.0.0.53:53: no such host
上述报错：树莓派现在没法解析网址（DNS解析失败)
1.修复树莓派的DNS设置
(1).打开配置文件：
```
sudo nano /etc/resolv.conf
```
(2).在文件的最开头添加这两行
```
nameserver 223.5.5.5
nameserver 8.8.8.8
```
保存退出： 按 Ctrl + O，回车，再按 Ctrl + X。
（这是阿里云和谷歌的公共拨号服务器，非常稳定）
2.测试网络是否通畅
输入以下命令，看看能不能得到回应：
```
ping -c 4 baidu.com
```
3.使用镜像：
我们换一个目前最稳的 DockerProxy 代理地址。
sudo docker pull docker.m.daocloud.io/homeassistant/home-assistant:stable
3.又失败了：
1)).代开配置文件
```
sudo nano /etc/docker/daemon.json
```
2)).清空内容，完整复制并粘贴以下这段
```Json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.unsee.tech",
    "https://docker.1panel.live",
    "https://mirror.baidubce.com"
  ]
}
```
3)).保存并退出
Ctrl + O -> Enter -> Ctrl + X。
4)).重启docker服务
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
5)).现在尝试最原始的命令（docker会自动遍历上面那几个加速器)
```
sudo docker pull homeassistant/home-assistant:stable
```

4.：启动Home Assistant
```
sudo docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Asia/Shanghai \
  -v /home/$USER/hass_config:/config \
  --network=host \
  docker.m.daocloud.io/homeassistant/home-assistant:stable
```

###### 第二步：修改名字并运行
等上面的进度条跑完后，我们给这个镜像起个好记的名字并启动它：
```
sudo docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Asia/Shanghai \
  -v /home/$USER/hass_config:/config \
  --network=host \
  hub-mirror.c.163.com/homeassistant/home-assistant:stable
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

### 关卡二： 把 Home Assistant 的“集装箱（容器）”放到这个地基上。
##### 试试这个：
删除报错的容器
```
sudo docker rm -f homeassistant
```

删除错误的镜像
```
sudo docker rmi homeassistant/home-assistant:stable
sudo docker rmi swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/homeassistant/home-assistant:stable
```
精准拉取“树莓派专用（arm64）”镜像

```
sudo docker pull --platform linux/arm64 swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/homeassistant/home-assistant:stable
```

正式运行：
```
sudo docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Asia/Shanghai \
  -v /home/$USER/hass_config:/config \
  --network=host \
  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/homeassistant/home-assistant:stable

```
```

###### 先删容器
```
sudo docker rm -f homeassistant
```
###### 删掉所有相关的镜像（根据你的列表把 ID 填进去，或者直接按名字删）
```
sudo docker rmi -f swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/homeassistant/home-assistant:stable
```
# 清理所有悬挂的镜像层
```
sudo docker system prune -a -f
```

既然华为云的这个地址让你反复抓取到 amd64，我们换一个专门为树莓派/ARM 优化的镜像仓库。

这次我们尝试 DaoCloud 的专用节点，它在处理多架构时通常更准确：

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



关闭ubuntu上的防火墙：
```
sudo ufw disable
```



```
esphome:
  name: esp32
  friendly_name: ESP32

esp32:
  board: esp32dev
  framework:
    type: esp-idf

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "YmjSICgiPKgGnTQ+m96vR5YJ3nrthFLrjquuEKKhqTo="

ota:
  - platform: esphome
    password: "842d8d2148edf77140f1416c5dd95aab"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Esp32 Fallback Hotspot"
    password: "JMEPtPOw7Z7M"

captive_portal:


remote_transmitter:
  pin: 4
  carrier_duty_percent: 50%

climate:
  - platform: midea_ir
    name: "1103_AC"

 
```
ubuntu@ubuntu-desktop:~/esphome-project$ uv run esphome -v run my_config.yaml
INFO ESPHome 2025.12.4
INFO Reading configuration my_config.yaml...
INFO Generating C++ source...
DEBUG Running to_code in esphome.components.network (num 14)
DEBUG Adding: // network:
DEBUG Adding: //   enable_ipv6: false
//   min_ipv6_addr_count: 0
DEBUG Adding define: USE_NETWORK=None
DEBUG Adding define: USE_NETWORK_IPV6=false
DEBUG  -> finished
DEBUG Running to_code in esphome.core.config (num 0)
DEBUG Adding: // esphome:
DEBUG Adding: //   name: test-esp32
//   min_version: 2025.12.4
//   build_path: build/test-esp32
//   friendly_name: ''
//   platformio_options: {}
//   environment_variables: {}
//   includes: []
//   includes_c: []
//   libraries: []
//   name_add_mac_suffix: false
//   debug_scheduler: false
//   areas: []
//   devices: []
DEBUG Adding global: using namespace esphome;
DEBUG Adding global: using std::isnan;
DEBUG Adding global: using std::min;
DEBUG Adding global: using std::max;
DEBUG Adding: App.pre_setup("test-esp32", "", "", __DATE__ ", " __TIME__, false);
DEBUG Adding define: ESPHOME_COMPONENT_COUNT=10
DEBUG Adding build flag: -fno-exceptions
DEBUG Adding build flag: -Wno-unused-variable
DEBUG Adding build flag: -Wno-unused-but-set-variable
DEBUG Adding build flag: -Wno-sign-compare
DEBUG  -> finished
DEBUG Running to_code in esphome.components.logger (num 2)
DEBUG Adding: // logger:
DEBUG Adding: //   id: logger_logger_id
//   baud_rate: 115200
//   tx_buffer_size: 512
//   deassert_rts_dtr: false
//   task_log_buffer_size: 768
//   hardware_uart: UART0
//   level: DEBUG
//   logs: {}
//   runtime_tag_levels: false
DEBUG Adding global: logger::Logger *logger_logger_id;
DEBUG Adding: logger_logger_id = new logger::Logger(115200, 512);
DEBUG Registered variable logger_logger_id of type logger::Logger
DEBUG Adding: logger_logger_id->create_pthread_key();
DEBUG Adding define: USE_ESPHOME_TASK_LOG_BUFFER=None
DEBUG Adding: logger_logger_id->init_log_buffer(768);
DEBUG Adding: logger_logger_id->set_log_level(ESPHOME_LOG_LEVEL_DEBUG);
DEBUG Adding: logger_logger_id->set_uart_selection(logger::UART_SELECTION_UART0);
DEBUG Adding: logger_logger_id->pre_setup();
DEBUG Adding define: USE_LOGGER=None
DEBUG Adding build flag: -DESPHOME_LOG_LEVEL=ESPHOME_LOG_LEVEL_DEBUG
DEBUG Adding: logger_logger_id->set_component_source(LOG_STR("logger"));
DEBUG Adding: App.register_component(logger_logger_id);
DEBUG  -> finished
DEBUG Running to_code in esphome.components.web_server_base (num 15)
DEBUG Adding: // web_server_base:
DEBUG Adding: //   id: web_server_base_webserverbase_id
DEBUG Adding global: web_server_base::WebServerBase *web_server_base_webserverbase_id;
DEBUG Adding: web_server_base_webserverbase_id = new web_server_base::WebServerBase();
DEBUG Registered variable web_server_base_webserverbase_id of type web_server_base::WebServerBase
DEBUG Adding: web_server_base_webserverbase_id->set_component_source(LOG_STR("web_server_base"));
DEBUG Adding: App.register_component(web_server_base_webserverbase_id);
DEBUG Adding: web_server_base::global_web_server_base = web_server_base_webserverbase_id;
DEBUG  -> finished
DEBUG Running to_code in esphome.components.captive_portal (num 8)
DEBUG Adding: // captive_portal:
DEBUG Adding: //   id: captive_portal_captiveportal_id
//   web_server_base_id: web_server_base_webserverbase_id
DEBUG Adding global: captive_portal::CaptivePortal *captive_portal_captiveportal_id;
DEBUG Adding: captive_portal_captiveportal_id = new captive_portal::CaptivePortal(web_server_base_webserverbase_id);
DEBUG Registered variable captive_portal_captiveportal_id of type captive_portal::CaptivePortal
DEBUG Adding: captive_portal_captiveportal_id->set_component_source(LOG_STR("captive_portal"));
DEBUG Adding: App.register_component(captive_portal_captiveportal_id);
DEBUG Adding define: USE_CAPTIVE_PORTAL=None
DEBUG  -> finished
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Adding: // wifi:
DEBUG Adding: //   ap:
//     ssid: Test-Esp32 Fallback Hotspot
//     password: xnHOF9FieYGW
//     id: wifi_wifiap_id
//     ap_timeout: 90s
//   id: wifi_wificomponent_id
//   domain: .local
//   reboot_timeout: 15min
//   power_save_mode: LIGHT
//   fast_connect: false
//   enable_btm: false
//   enable_rrm: false
//   passive_scan: false
//   enable_on_boot: true
//   min_auth_mode: WPA2
//   networks:
//     - ssid: cmd1122
//       password: 01256789
//       id: wifi_wifiap_id_2
//       priority: 0
//   use_address: test-esp32.local
DEBUG Adding global: wifi::WiFiComponent *wifi_wificomponent_id;
DEBUG Adding: wifi_wificomponent_id = new wifi::WiFiComponent();
DEBUG Registered variable wifi_wificomponent_id of type wifi::WiFiComponent
DEBUG Adding: wifi_wificomponent_id->set_use_address("test-esp32.local");
DEBUG Adding: wifi_wificomponent_id->init_sta(1);
DEBUG Adding: {
DEBUG Adding: wifi::WiFiAP wifi_wifiap_id_2 = wifi::WiFiAP();
DEBUG Registered variable wifi_wifiap_id_2 of type wifi::WiFiAP
DEBUG Adding: wifi_wifiap_id_2.set_ssid("cmd1122");
DEBUG Adding: wifi_wifiap_id_2.set_password("01256789");
DEBUG Adding: wifi_wifiap_id_2.set_priority(0);
DEBUG Adding: wifi_wificomponent_id->add_sta(wifi_wifiap_id_2);
DEBUG Adding: }
DEBUG Adding: {
DEBUG Adding: wifi::WiFiAP wifi_wifiap_id = wifi::WiFiAP();
DEBUG Registered variable wifi_wifiap_id of type wifi::WiFiAP
DEBUG Adding: wifi_wifiap_id.set_ssid("Test-Esp32 Fallback Hotspot");
DEBUG Adding: wifi_wifiap_id.set_password("xnHOF9FieYGW");
DEBUG Adding: wifi_wificomponent_id->set_ap(wifi_wifiap_id);
DEBUG Adding: }
DEBUG Adding: wifi_wificomponent_id->set_ap_timeout(90000);
DEBUG Adding define: USE_WIFI_AP=None
DEBUG Adding: wifi_wificomponent_id->set_reboot_timeout(900000);
DEBUG Adding: wifi_wificomponent_id->set_power_save_mode(wifi::WIFI_POWER_SAVE_LIGHT);
DEBUG Adding: wifi_wificomponent_id->set_min_auth_mode(wifi::WIFI_MIN_AUTH_MODE_WPA2);
DEBUG Adding define: USE_WIFI=None
DEBUG Adding: wifi_wificomponent_id->set_component_source(LOG_STR("wifi"));
DEBUG Adding: App.register_component(wifi_wificomponent_id);
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.mdns (num 16)
DEBUG Adding: // mdns:
DEBUG Adding: //   id: mdns_mdnscomponent_id
//   disabled: false
//   services: []
DEBUG Adding define: USE_MDNS=None
DEBUG Adding define: MDNS_SERVICE_COUNT=1
DEBUG Adding global: mdns::MDNSComponent *mdns_mdnscomponent_id;
DEBUG Adding: mdns_mdnscomponent_id = new mdns::MDNSComponent();
DEBUG Registered variable mdns_mdnscomponent_id of type mdns::MDNSComponent
DEBUG Adding: mdns_mdnscomponent_id->set_component_source(LOG_STR("mdns"));
DEBUG Adding: App.register_component(mdns_mdnscomponent_id);
DEBUG  -> finished
DEBUG Running to_code in esphome.components.ota (num 4)
DEBUG Adding: // ota:
DEBUG Adding define: USE_OTA=None
DEBUG  -> finished
DEBUG Running to_code in esphome.components.esphome.ota (num 6)
DEBUG Adding: // ota.esphome:
DEBUG Adding: //   platform: esphome
//   password: ''
//   id: esphome_esphomeotacomponent_id
//   version: 2
//   port: 3232
DEBUG Adding global: esphome::ESPHomeOTAComponent *esphome_esphomeotacomponent_id;
DEBUG Adding: esphome_esphomeotacomponent_id = new esphome::ESPHomeOTAComponent();
DEBUG Registered variable esphome_esphomeotacomponent_id of type esphome::ESPHomeOTAComponent
DEBUG Adding: esphome_esphomeotacomponent_id->set_port(3232);
DEBUG Adding define: USE_OTA_VERSION=2
DEBUG Adding: esphome_esphomeotacomponent_id->set_component_source(LOG_STR("esphome.ota"));
DEBUG Adding: App.register_component(esphome_esphomeotacomponent_id);
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.esphome.ota (num 6)
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.web_server.ota (num 5)
DEBUG Adding: // ota.web_server:
DEBUG Adding: //   platform: web_server
//   id: web_server_webserverotacomponent_id
DEBUG Adding global: web_server::WebServerOTAComponent *web_server_webserverotacomponent_id;
DEBUG Adding: web_server_webserverotacomponent_id = new web_server::WebServerOTAComponent();
DEBUG Registered variable web_server_webserverotacomponent_id of type web_server::WebServerOTAComponent
DEBUG Running to_code in esphome.components.esphome.ota (num 6)
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.web_server.ota (num 5)
DEBUG Running to_code in esphome.components.esphome.ota (num 6)
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.web_server.ota (num 5)
DEBUG Running to_code in esphome.components.esphome.ota (num 6)
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG Running to_code in esphome.components.safe_mode (num 11)
DEBUG Adding: // safe_mode:
DEBUG Adding: //   id: safe_mode_safemodecomponent_id
//   boot_is_good_after: 1min
//   disabled: false
//   num_attempts: 10
//   reboot_timeout: 5min
DEBUG Adding global: safe_mode::SafeModeComponent *safe_mode_safemodecomponent_id;
DEBUG Adding: safe_mode_safemodecomponent_id = new safe_mode::SafeModeComponent();
DEBUG Registered variable safe_mode_safemodecomponent_id of type safe_mode::SafeModeComponent
DEBUG Adding: safe_mode_safemodecomponent_id->set_component_source(LOG_STR("safe_mode"));
DEBUG Adding: App.register_component(safe_mode_safemodecomponent_id);
DEBUG Adding: if (safe_mode_safemodecomponent_id->should_enter_safe_mode(10, 300000, 60000)) return;
DEBUG  -> finished
DEBUG Running to_code in esphome.components.web_server.ota (num 5)
DEBUG Adding: web_server_webserverotacomponent_id->set_component_source(LOG_STR("web_server.ota"));
DEBUG Adding: App.register_component(web_server_webserverotacomponent_id);
DEBUG Adding define: USE_WEBSERVER_OTA=None
DEBUG  -> finished
DEBUG Running to_code in esphome.components.esphome.ota (num 6)
DEBUG  -> finished
DEBUG Running to_code in esphome.components.wifi (num 7)
DEBUG  -> finished
DEBUG Running to_code in esphome.components.api (num 3)
DEBUG Adding: // api:
DEBUG Adding: //   password: ''
//   id: api_apiserver_id
//   port: 6053
//   reboot_timeout: 15min
//   batch_delay: 100ms
//   custom_services: false
//   homeassistant_services: false
//   homeassistant_states: false
//   listen_backlog: 4
//   max_connections: 8
//   max_send_queue: 8
DEBUG Adding global: api::APIServer *api_apiserver_id;
DEBUG Adding: api_apiserver_id = new api::APIServer();
DEBUG Registered variable api_apiserver_id of type api::APIServer
DEBUG Adding: api_apiserver_id->set_component_source(LOG_STR("api"));
DEBUG Adding: App.register_component(api_apiserver_id);
DEBUG Adding: api_apiserver_id->set_port(6053);
DEBUG Adding: api_apiserver_id->set_reboot_timeout(900000);
DEBUG Adding: api_apiserver_id->set_batch_delay(100);
DEBUG Adding: api_apiserver_id->set_listen_backlog(4);
DEBUG Adding: api_apiserver_id->set_max_connections(8);
DEBUG Adding define: API_MAX_SEND_QUEUE=8
DEBUG Adding define: USE_API_PLAINTEXT=None
DEBUG Adding define: USE_API=None
DEBUG Adding global: using namespace api;
DEBUG  -> finished
DEBUG Running _add_automations in esphome.core.config (num 20)
DEBUG  -> finished
DEBUG Running to_code in esphome.components.esp32 (num 1)
DEBUG Adding: // esp32:
DEBUG Adding: //   board: esp32dev
//   framework:
//     type: esp-idf
//     version: 5.5.1
//     sdkconfig_options: {}
//     log_level: ERROR
//     advanced:
//       compiler_optimization: SIZE
//       enable_idf_experimental_features: false
//       enable_lwip_assert: true
//       ignore_efuse_custom_mac: false
//       ignore_efuse_mac_crc: false
//       enable_lwip_mdns_queries: true
//       enable_lwip_bridge_interface: false
//       enable_lwip_tcpip_core_locking: true
//       enable_lwip_check_thread_safety: true
//       disable_libc_locks_in_iram: true
//       disable_vfs_support_termios: true
//       disable_vfs_support_select: true
//       disable_vfs_support_dir: true
//       freertos_in_iram: false
//       ringbuf_in_iram: false
//       execute_from_psram: false
//       loop_task_stack_size: 8192
//     components: []
//     platform_version: https:github.com/pioarduino/platform-espressif32/releases/download/55.03.31-2/platform-espressif32.zip
//     source: pioarduino/framework-espidf@https:github.com/pioarduino/esp-idf/releases/download/v5.5.1/esp-idf-v5.5.1.tar.xz
//   flash_size: 4MB
//   variant: ESP32
//   cpu_frequency: 160MHZ
DEBUG Adding build unflag: -std=gnu++11
DEBUG Adding build unflag: -std=gnu++14
DEBUG Adding build unflag: -std=gnu++17
DEBUG Adding build unflag: -std=gnu++23
DEBUG Adding build unflag: -std=gnu++2a
DEBUG Adding build unflag: -std=gnu++2b
DEBUG Adding build unflag: -std=gnu++2c
DEBUG Adding build flag: -std=gnu++20
DEBUG Adding build flag: -DUSE_ESP32
DEBUG Adding build flag: -Wl,-z,noexecstack
DEBUG Adding define: ESPHOME_BOARD="esp32dev"
DEBUG Adding build flag: -DUSE_ESP32_VARIANT_ESP32
DEBUG Adding define: ESPHOME_VARIANT="ESP32"
DEBUG Adding define: ESPHOME_THREAD_MULTI_ATOMICS=None
DEBUG Adding build flag: -DUSE_ESP_IDF
DEBUG Adding build flag: -DUSE_ESP32_FRAMEWORK_ESP_IDF
DEBUG Adding build flag: -Wno-nonnull-compare
INFO Setting CONFIG_LWIP_MAX_SOCKETS to 11 (registered: api=4, captive_portal=4, mdns=2, ota=1)
DEBUG Adding define: ESPHOME_LOOP_TASK_STACK_SIZE=8192
DEBUG Adding define: USE_ESP_IDF_VERSION_CODE=VERSION_CODE(5, 5, 1)
DEBUG  -> finished
DEBUG Running to_code in esphome.components.preferences (num 9)
DEBUG Adding: // preferences:
DEBUG Adding: //   id: preferences_intervalsyncer_id
//   flash_write_interval: 60s
DEBUG Adding global: preferences::IntervalSyncer *preferences_intervalsyncer_id;
DEBUG Adding: preferences_intervalsyncer_id = new preferences::IntervalSyncer();
DEBUG Registered variable preferences_intervalsyncer_id of type preferences::IntervalSyncer
DEBUG Adding: preferences_intervalsyncer_id->set_write_interval(60000);
DEBUG Adding: preferences_intervalsyncer_id->set_component_source(LOG_STR("preferences"));
DEBUG Adding: App.register_component(preferences_intervalsyncer_id);
DEBUG  -> finished
DEBUG Running to_code in esphome.components.md5 (num 10)
DEBUG Adding: // md5:
DEBUG Adding define: USE_MD5=None
DEBUG  -> finished
DEBUG Running to_code in esphome.components.socket (num 12)
DEBUG Adding: // socket:
DEBUG Adding: //   implementation: bsd_sockets
DEBUG Adding define: USE_SOCKET_IMPL_BSD_SOCKETS=None
DEBUG Adding define: USE_SOCKET_SELECT_SUPPORT=None
DEBUG  -> finished
DEBUG Running to_code in esphome.components.sha256 (num 13)
DEBUG Adding: // sha256:
DEBUG Adding: //   {}
DEBUG  -> finished
DEBUG Running to_code in esphome.components.web_server_idf (num 17)
DEBUG Adding: // web_server_idf:
DEBUG Adding: //   {}
DEBUG  -> finished
DEBUG Running _add_platform_defines in esphome.core.config (num 18)
DEBUG  -> finished
DEBUG Running _add_controller_registry_define in esphome.core.config (num 19)
DEBUG Adding define: USE_CONTROLLER_REGISTRY=None
DEBUG Adding define: CONTROLLER_REGISTRY_MAX=1
DEBUG  -> finished
DEBUG Running final_step in esphome.components.logger (num 21)
DEBUG  -> finished
DEBUG Running final_step in esphome.components.wifi (num 22)
DEBUG  -> finished
INFO Compiling app... Build path: /home/ubuntu/esphome-project/.esphome/build/test-esp32
DEBUG Running:  platformio run -d /home/ubuntu/esphome-project/.esphome/build/test-esp32 -v
Processing test-esp32 (board: esp32dev; board_build.partitions: partitions.csv; board_upload.flash_size: 4MB; board_upload.maximum_size: 4194304; build_flags: -DESPHOME_LOG_LEVEL=ESPHOME_LOG_LEVEL_DEBUG, -DUSE_ESP32, -DUSE_ESP32_FRAMEWORK_ESP_IDF, -DUSE_ESP32_VARIANT_ESP32, -DUSE_ESP_IDF, -Wl,-z,noexecstack, -Wno-nonnull-compare, -Wno-sign-compare, -Wno-unused-but-set-variable, -Wno-unused-variable, -fno-exceptions, -std=gnu++20; build_unflags: -std=gnu++11, -std=gnu++14, -std=gnu++17, -std=gnu++23, -std=gnu++2a, -std=gnu++2b, -std=gnu++2c; extra_scripts: pre:pre_build.py, post:post_build.py, pre:cxx_flags.py; framework: espidf; lib_compat_mode: strict; lib_deps: ; lib_ldf_mode: off; platform: https://github.com/pioarduino/platform-espressif32/releases/download/55.03.31-2/platform-espressif32.zip; platform_packages: pioarduino/framework-espidf@https://github.com/pioarduino/esp-idf/releases/download/v5.5.1/esp-idf-v5.5.1.tar.xz)
-------------------------------------------------------------------------------------------------------------------------------------------
DEBUG Version comparison: installed='5.3.4' vs required='5.3.4'
DEBUG tool-esp_install version 5.3.4 is already correctly installed
DEBUG tool-esp_install is available and ready
DEBUG Tool tool-esptoolpy found with correct version
Error: Failed to install Python dependencies (exit code: 2)
Error: Failed to install Python dependencies into penv
```

```
esphome:
  name: test-esp32

esp32:
  board: esp32dev
  framework:
    type: esp-idf

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: ""

ota:
  - platform: esphome
    password: ""

wifi:
  ssid: "cmd1122"
  password: "01256789"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Test-Esp32 Fallback Hotspot"
    password: "xnHOF9FieYGW"

captive_portal:
```
