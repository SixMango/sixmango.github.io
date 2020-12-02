---
layout: post
title: TIM个人文件夹设置
date: 2020-11-15 14:13:44
tags:
- tim
categories: 
- 生活
---

QQ轻聊版、TIM版有个BUG：

如果电脑上之前没有安装过QQ完整版（比如我新装的虚拟机），那么QQ轻聊版、TIM版里面，设置修改个人文件夹位置将无法生效。在QQ轻聊版、TIM版设置界面修改个人文件夹位置后，会提示转移数据文件，之后会重启轻聊版/TIM，但重启后的登录界面上没有之前登录过的QQ号，重新输入QQ号和密码登录后，查看配置界面，发现个人文件夹位置还是修改之前的默认路径（通常是在我的文档下）。

1. 打开轻聊版/TIM设置界面，找到个人文件夹设置的地方，点击“打开个人文件夹”按钮，会打开当前的个人文件夹，完全退出轻聊版/TIM。
2. 到个人文件夹的上一层目录，把里面的QQ号码文件夹和All Users文件夹移动或复制到新的个人文件夹位置，比如**D:\MyQQData**目录下。
3. 进入"C:\Users\Public\Documents\Tencent\QQ"目录（如果没有这个目录，请自行创建），在该目录下右键新建文本文件，并重命名为“UserDataInfo.ini”
4. 双击打开“UserDataInfo.ini”文件，将下列内容复制粘贴进去，**UserDataSavePath=D:\MyQQData** 这里就是指定新的个人文件夹位置，保存并关闭。

```
[UserDataImportSet]
NeedImport=0
OldVersion=
OldVerDataPathType=
OldVerDataPath=
OldQQInstallPath=C:\Program Files (x86)\Tencent\QQ
[UserDataSet]
UserDataSavePathType=2
UserDataSavePath=D:\MyQQData
NewVersion=
```



5. 重新启动轻聊版/TIM，打开配置界面，看到修改已经生效了。  