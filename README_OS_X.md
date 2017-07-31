# Modifying for OS X

This guide explains how to use OS X's built in tools to capture a 4 way handshake and use (a modified version of) naive-hashcat to determine the password of a WPA/WPA2 wireless network.

__DISCLAIMER: This tutorial is for educational purposes only. Use only on networks you own. Author is not responsible for its use.__

## Getting Started

This tutorial assumes that you:

- Have a general comfortability using the command-line
- Are running OS X 10.12 (though older versions should also work)
- Have a wireless card that supports [monitor mode](https://en.wikipedia.org/wiki/Monitor_mode) (I used a MacBook Pro Mid-2015)

## Steps

Luckily, OS X comes with a suite of wireless diagnostic tools. To open them, hold down the option key on your keyboard and click on the Wi-Fi icon in the menu bar. Then click "Open Wireless Diagnostics..."

### Determine the channel of your target network

With Wireless Diagnostics open, click on Window > Scan. Find the target network, note its channel and width.

### Capture a 4-way Handshake
1. With Wireless Diagnostics open, click on Window > Sniffer. Select the channel and width that you found in the previous step.
2. Now you'll need to wait for a device to connect to the target network. If you have a different device that can connect to the network, do that now to capture the handshake.
3. Once you think you've got a handshake, click Stop.
4. The .wcap capture file will be in `/var/tmp/`.
5. Convert the capture file by uploading it to https://hashcat.net/cap2hccapx/. If you captured any handshakes, the site will start downloading a .hccapx file.

### Running naive-hashcat
1. Download naive-hashcat:

```bash
# download
git clone https://github.com/brannondorsey/naive-hashcat
cd naive-hashcat

# download the 134MB rockyou dictionary file
curl -L -o dicts/rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

2. Since the included binary for hashcat won't work on OS X, we'll need to grab and compile our own version. While in the `naive-hashcat` directory run
```
git clone https://github.com/hashcat/hashcat.git
mkdir -p hashcat/deps
git clone https://github.com/KhronosGroup/OpenCL-Headers.git hashcat/deps/OpenCL
cd hashcat/
make
./example0.sh # just to make sure hashcat is working, once it starts going through hashes you can quit
cd ..
```

3. Now we'll need to modify naive-hashcat to look our new version of hashcat. Open naive-hashcat.sh with any text editor. Replace this:
```
if [ $(uname -m) == 'x86_64' ]; then
	HASHCAT="./hashcat-3.6.0/hashcat64.bin"
else
	HASHCAT="./hashcat-3.6.0/hashcat32.bin"
fi
```

with this
```
if [ $(uname -m) == 'x86_64' ]; then
	HASHCAT="./hashcat/hashcat"
else
	HASHCAT="./hashcat/hashcat"
fi
```

4. Finally, now we can run naive-hashcat.sh. Change `handshake.hccapx` to the name of the file you received earlier. `out.pot` is the name of the output file. 
```
HASH_FILE=handshake.hccapx POT_FILE=out.pot HASH_TYPE=2500 ./naive-hashcat.sh
```

## Attribution
Thanks to [this page describing on how use hashcat on OS X](https://www.phillips321.co.uk/2016/07/09/hashcat-on-os-x-getting-it-going/)
