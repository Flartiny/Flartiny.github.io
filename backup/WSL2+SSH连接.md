写于体验Windsurf时，其连接WSL2和vscode的远程连接功能略有区别
## 基础配置
在WSL2中编辑
vim /etc/ssh/sshd_config
```
Port 22
ListenAddress 0.0.0.0

PermitRootLogin yes

PasswordAuthentication yes
```
编辑完成后重启ssh服务，以后出现连接不上的问题也可以尝试重启该服务
```
sudo service ssh restart
```

还可以修改C盘User目录下.wslconfig文件
```
networkingMode=mirrored # 开启镜像网络
```
以便以如下配置连接WSL2
```
Host 127.0.0.1
  HostName 127.0.0.1
  User root
  Port 22
```
---
## 免密连接
本机生成密钥对
```
ssh-keygen
```
进入C盘User目录下.ssh目录
将id_rsa.pub文件复制到WSL2的/root/.ssh目录下
```
chmod 700 .ssh/
cd .ssh/
cat id_rsa.pub >> authorized_keys 
chmod 600 authorized_keys
sudo service ssh restart
```
