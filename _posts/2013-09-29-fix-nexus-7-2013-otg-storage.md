---
layout: post
title: "Fix: Nexus 7(2013) Wifi OTG Storage Issue"
description: "解决二代N7无法使用OTG存储设备的问题"
category: 
tags: [Nexus, OTG]
---
{% include JB/setup %}

需要安装Nexus Photo Viewer([Google Play](https://play.google.com/store/apps/details?id=com.homeysoft.nexususb.viewer))，依据软件开发者的博客([blogspot](http://nexususb.blogspot.com/))提供的解决方案即可解决。

下面是软件开发者提供的解决步骤：

1. Hold the device in Portrait Mode（竖屏）
2. Open Android Settings and make the following __temporary__ changes（以下步骤修改的配置，可在问题解决后恢复）
3. Under Security -> Lock Screen.  Select None（取消密码、图案锁屏等）
4. Under Accessibility -> Auto-rotate screen.  Make sure it is NOT checked（取消自动旋转屏幕）
5. __(Nexus 7 2013 Only)__ Under Language and input -> English (United States)（__语言改为美式英文__）
6. Open NMI/NPV manually via the icon.（打开Nexus Photo Viewer）
7. Connect the OTG and Flash/Pen Drive（连接OTS存储设备）
8. Press and Hold Power until you see "Power Off"（关机）
9. Tap OK to Power Off
10. Power On（开机）
11. Repeat steps 8-10 until you see this prompt or NMI opens automatically.  If you have repeated this more than 5 times see below.\*（重复8～10，直到Nexus Photo Viewer开机自动打开，并且可以浏览OTG存储设备内容，软件开发者博客提供了效果图）

---

2013-10-10 更新

按照上述方法虽然能够解决二代N7的OTG存储设备使用问题，但是会导致机器__极其不稳定__。

这种不稳定将直接导致机器__重启__，并在重启之后恢复正常，当然OTG功能又不见了……