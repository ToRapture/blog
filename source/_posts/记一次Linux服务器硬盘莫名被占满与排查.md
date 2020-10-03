---
title: 记一次Linux服务器硬盘莫名被占满与排查
date: 2017-08-29 14:14:00
tags: 问题排查
---

<p>远程连接Mysql时发生错误，提示为ERROR 1030 (HY000): Got error 28 from storage engine.</p>
<p>Google这个错误说是由于硬盘满了，连接到服务器上查了一下发现果然是这样。</p>
<p><img src="http://images2017.cnblogs.com/blog/1224734/201708/1224734-20170829140808046-301674445.png" alt="" /></p>
<p>&nbsp;</p>
<p>然后开始猜测发生的原因。</p>
<p>之前的几个月服务器上只运行着Web应用，而且很稳定，近期的变动只有新安装了polipo，怎么配置的忘了。</p>
<p>首先猜到的是Mysql相关的文件和其他应用的日志文件。</p>
<p>在根目录下执行sudo du -s -h ./*，发现/var占用空间很多，</p>
<p><img src="http://images2017.cnblogs.com/blog/1224734/201708/1224734-20170829141206874-19035931.png" alt="" /></p>
<p>再进入/var/中定位，发现/var/cache/占用空间很多，</p>
<p>sudo apt-get remove --purge polipo卸载polipo之后磁盘空间恢复，具体为什么polipo占了这么多空间我也不知道，可能是因为使用的姿势有问题？</p>