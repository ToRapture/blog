---
title: Ubuntu更改鼠标滚轮速度
date: 2020-10-04 15:12:25
tags:
- Ubuntu
---

## 安装 imwheel

`$ sudo apt-get install imwheel`

## 编辑imwheelrc配置

`$ vim ~/.imwheelrc`

```
".*"
None,      Up,   Button4, 2
None,      Down, Button5, 2
Control_L, Up,   Control_L|Button4
Control_L, Down, Control_L|Button5
Shift_L,   Up,   Shift_L|Button4
Shift_L,   Down, Shift_L|Button5
```

## 配置开机启动

`$ gnome-session-properties`

![](/images/posts/Ubuntu更改鼠标滚轮速度/0.png)