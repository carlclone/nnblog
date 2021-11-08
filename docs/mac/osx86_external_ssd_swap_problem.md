# 外部SSD macOS的SWAP分区开启问题

## 问题
公司电脑只有8G内存,打开大量网页的时候爆内存,黑苹果卡住

## 原因
由于自己的黑苹果安装在外接的SSD硬盘上,通过USB接口连接,在mac中被识别为External Storage , 因此无法mount swap分区 , 相当于模式1 , 只使用内存 (为什么没有 OOM Killer 机制) , 一爆内存就死机

## 解决
开机的时候强制mount ,添加启动脚本到/Library/LaunchDaemons/

![image-20191120140931373](/img/osx86_swap_sloved.jpg)

### 参考 Stack Exchange
https://apple.stackexchange.com/questions/318266/mac-os-x-is-not-creating-a-swap-file

```
I had the same problem when I installed High Sierra on external SSD.
Volume disk3s4 647DA4A9-7E85-4523-A4D2-F0392D3789D4
        ---------------------------------------------------
        APFS Volume Disk (Role):   disk3s4 (VM)
        Name:                      VM (Case-insensitive)
        Mount Point:               Not Mounted
        Capacity Consumed:         4294987776 B (4.3 GB)
        FileVault:                 No
Solution:
1. Create a plist file as root user and put it in /Library/LaunchDaemons/ folder. It has to be written in reverse domain notation like this: /Library/LaunchDaemons/local.mountdisk3s4.plist
2.  
3. Just copy this xml data in your plist file and change the name of APFS VM Volume with yours. <?xml version="1.0" encoding="UTF-8"?>
4. <!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
5. <plist version="1.0">
6. <dict>
7.      <key>Label</key>
8.      <string>THE NAME OF FILE</string>
9.      <key>ProgramArguments</key>
10.      <array>
11.           <string>/sbin/mount_apfs</string>
12.           <string>YOUR APFS VOLUME</string>
13.           <string>/private/var/vm</string>
14.      </array>
15.      <key>KeepAlive</key>
16.      <dict>
17.     <key>SuccessfulExit</key>
18.     <false/>
19.      </dict>    
20. </dict>
21. </plist>
22.  In my case it looks like this: <?xml version="1.0" encoding="UTF-8"?>
23. <!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
24. <plist version="1.0">
25. <dict>
26.      <key>Label</key>
27.      <string>local.mountdisk3s4</string>
28.      <key>ProgramArguments</key>
29.      <array>
30.           <string>/sbin/mount_apfs</string>
31.           <string>disk3s4</string>
32.           <string>/private/var/vm</string>
33.      </array>
34.      <key>KeepAlive</key>
35.      <dict>
36.     <key>SuccessfulExit</key>
37.     <false/>
38.      </dict>    
39. </dict>
40. </plist>
41.  
42. Reboot your Mac 
```

## 彩蛋

附上一份macOS的成长历史

|Version |Codename |﻿Most recent version |
|--- |--- |--- |
|Mac OS X 10.0 |Cheetah |﻿10.0.4 (June 22, 2001) |
|Mac OS X 10.1 |Puma |10.1.5 (June 6, 2002) |
|Mac OS X 10.2 |Jaguar |10.2.8 (October 3, 2003) |
|Mac OS X 10.3 |Panther |10.3.9 (April 15, 2005) |
|Mac OS X 10.4 |Tiger |10.4.11 (November 14, 2007) |
|Mac OS X 10.5 |Leopard |10.5.8 (August 5, 2009) |
|Mac OS X 10.6 |Snow Leopard |10.6.8 v1.1 (July 25, 2011) |
|Mac OS X 10.7 |Lion |10.7.5 (September 19, 2012) |
|OS X 10.8 |Mountain Lion |10.8.5 (12F45) (October 3, 2013) |
|OS X 10.9 |Mavericks |10.9.5 (13F1112) (September 18, 2014)[170] |
|OS X 10.10 |Yosemite |10.10.5 (14F27) (August 13, 2015) |
|OS X 10.11 |El Capitan |10.11.6 (15G31) (July 18, 2016) |
|macOS 10.12 |Sierra |10.12.6 (16G29) (July 19, 2017) |
|macOS 10.13 |High Sierra |10.13.6 (17G65) (July 9, 2018) |
|macOS 10.14 |Mojave |10.14 (18A391) (September 24, 2018) |