Teamspeak可以部署在云服务器上为使用者提供语音等服务，而且不会占用太多资源

由于网络因素，推荐本地下载后上传到云服务器
下载地址 https://teamspeak.com/zh-CN/downloads/#server

```shell
// 上传后解压
tar -xvf filename
// 重命名
mv file teamspeak
// 移动到home目录
mv teamspeak /home
cd /home/teamspeak
// 创建teamspeak用户
useradd teamspeak
passwd teamspeak
// 赋予权限
su root
chown -R teamspeak:teamspeak /home/teamspeak/
// 切换到teamspeak用户
su teamspeak
```

如果切换到teamspeak用户后，界面只显示$，请输入bash后回车即可临时解决
如果想永久解决，切换至root用户后，vim打开/etc/passwd文件，将最后一行的sh改为bash保存即可

```shell
// 进入teamspeak目录，创建授权文件，初始化
touch .ts3server_license_accepted
./ts3server_startscript.sh start
```

控制台反馈的token等信息建议复制保存在文件中，初次登录时或者未来出现特殊情况会用到

```shell
// 端口管理
sudo apt install ufw -y
ufw enable
ufw allow ssh
ufw allow 9987/udp
ufw allow 30033/tcp
ufw status #查看ufw状态
// 也可以在服务器商处对端口放行
```

![alt text](https://pub-d3faa1947eb448819cde832afc98290f.r2.dev/blog/2024/08/4aaf4783ffeb38eeb908dab3ddad49db.jpg)

```shell
// 接下来是进程守护
vim /lib/systemd/system/teamspeak.service
```

**teamspeak.service**中填入以下内容
如果之前路径不同的话，以下内容相应部分自行更改

```service
[Unit]
Description=teamspeak
After=network.target

[Service]
User=teamspeak
Group=teamspeak
Type=forking
WorkingDirectory=/home/teamspeak/
PIDFile=/home/teamspeak/ts3server.pid
ExecStart=/home/teamspeak/ts3server_startscript.sh start
ExecStop=/home/teamspeak/ts3server_startscript.sh stop
RestartSec=15
Restart=always

[Install]
WantedBy=multi-user.target
```

```shell
重启systemd
systemctl daemon-reload
// 开机自启动等操作
systemctl enable teamspeak.service
systemctl start teamspeak.service
systemctl restart teamspeak.service
systemctl stop teamspeak.service
// 如果进程守护出错，先停止ts服务再重试
```

然后本机使用客户端连接ip就行了，第一次会要求输入token，
之后可以点击'工具-身份'将身份状态导出到本地

---

参考原文 https://blog.im.ci/study-notes/linux-notes/468/
汉化包和客户端资源 https://teamspeak.app/docs/

之后可能补充ts机器人相关...
