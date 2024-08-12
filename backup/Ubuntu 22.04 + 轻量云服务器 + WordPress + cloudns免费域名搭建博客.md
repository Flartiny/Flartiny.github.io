一时兴起使用了基于github pages的博客后，想再整点花里胡哨的。正好手头有闲置的轻量云服务器，于是想先试试看部署一个基于WordPress的个人博客，之后换服务器的时候不至于折腾太久。事实上这次也已经踩了很多坑，好在应该是得到了完整的成果。

---
那么XShell连接云服务器后直接开始操作

    // 更新软件包
    sudo apt update
    sudo apt upgrade

    // apache2
    sudo apt install apache2

    // 接下来是一些检验操作 其实可以跳过
    // 查看状态 active即可
    sudo systemctl status apache2
    // 应当返回enabled
    sudo systemctl is-enabled apache2
    // 得到第三行的ip段本机访问可以看到apache运行，不重要
    ip addr | grep inet

    // mariadb
    sudo apt install mariadb-server mariadb-client

    // 同样查看状态
    sudo systemctl status mariadb.service
    sudo systemctl is-enabled mariadb.service

    // 应该是安全服务 按照提示输完密码后
    // 提示safely answer 'n'的就直接n，其他都y
    sudo mysql_secure_installation
    // php
    sudo apt install php libapache2-mod-php php-mysql

    // 安装完成后
    sudo systemctl restart apache2
    sudo vim /var/www/html/info.php

往**info.php**中写入以下内容

```php
<?php
 phpinfo();
?>
```

    // 访问之前获取的ip/info.php应该能看到相关内容，不过不重要
    // phpmyadmin 最后弹出的configuration要选择apache2
    // 然后后面yes、设置管理员密码之类的东西
    sudo apt install phpmyadmin

![image1](https://imgos.cn/2024/08/12/66b9913b27249.png)

    sudo systemctl restart apache2.service
    // 访问ip/phpmyadmin进入页面可以登录管理员账户(phpmyadmin;设置的密码)

    sudo ufw enable
    sudo ufw allow 'Apache Full'
    // 可选
    sudo ufw allow OpenSSH

    // wordpress
    wget -c http://wordpress.org/latest.tar.gz
    tar -xzvf latest.tar.gz
    // 这里最后文件夹可以自己命名
    sudo cp -R wordpress /var/www/html/flartiny.com
    // 检查权限
    ls -l /var/www/html
    sudo chown -R www-data:www-data /var/www/html/flartiny.com
    sudo chown -R 775 /var/www/html/flartiny.com

    // database
    sudo mysql -u root -p
    // 建立数据库，名字自己取
    CREATE DATABASE flartiny;
    // 登录后操作，语句末尾记得加分号
    // 这里username和password自己改，是后续操作用到的用户
    // CREATE USER 'username'@'%' IDENTIFIED BY 'password';
    CREATE USER 'walu'@'%' IDENTIFIED BY 'pass246';
    // GRANT ALL PRIVILEGES ON *.* TO 'username'@'%' WITH GRANT OPTION;
    GRANT ALL PRIVILEGES ON *.* TO 'walu'@'%' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    exit

    // 修改wordpress配置文件
    sudo mv wp-config-sample.php wp-config.php
    sudo vim wp-config.php

修改**wp-config.php**中的一些内容

```
DB_NAME后参数改为数据库名(flartiny)
DB_USER后参数改为用户名(walu)
DB_PASSWORD后参数改为用户密码(pass246)
```
    // .conf前名字按之前的改
    sudo vim /etc/apache2/sites-available/flartiny.com.conf

**flartiny.com.conf**中按以下模板填入，几个用户名的地方自己改一下

```
<VirtualHost *.80>
 ServerName flartiny.com
 ServerAdmin walu@localhost
 DocumentRoot /var/www/html/flartiny.com
 ErrorLog ${APACHE_LOG_DIR}/error.log
 CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

    // 语法检查
    apachectl -t

    sudo vim /etc/apache2/apache2.conf

**apache2.conf**末尾填入

```
#Add ServerName
ServerName localhost
```

    apachectl -t
    // 名字按之前自己的改
    sudo a2ensite flartiny.com.conf
    systemctl reload apache2
    sudo a2dissite 000-default.conf
    sudo service apache2 restart

到这里服务器端的配置就告一段落了
可以访问服务器**ip/wp-admin/install.php**初始化安装wordpress，生成管理员账户

---

cloudns注册账号后免费计划可以获得一个免费域名,以我的为例(```flartiny.cloudns.ch```)
随后的步骤如下：

1. 在cloudflare'**网站**'标签处选择添加站点，填入```flartiny.cloudns.ch```
2. 过程中有两个名称服务器，需要在cloudns中先配置两条NS记录，将名称服务器填入
3. 等待自动配置完成后转到'**SSL/TLS**'标签下的'概述'，加密模式我这里用的是'**灵活**'
4. 转到'**SSL/TLS**'标签下的'**边缘证书**'，用证书里的两组信息在cloudns中创建TXT记录
5. 在cloudns中,添加NS记录也指向之前的名称服务器，'**主机**'处填'**blog**'(或者其他也行)
6. 在cloudflare转到'**DNS**'标签，添加A类型记录名称为'**blog**'与上面对应，指向服务器ip
7. 等待证书等内容生效

![image2](https://imgos.cn/2024/08/12/66b9913ba44b0.png)

---

最后还需要有一些操作，也是我不太确定的部分，其中可能有不太准确或者多余的步骤
总之先讲如何操作：

1. 进入wordpress管理面板插件，搜索安装**cloudflare插件**
2. 在cloudflare转到'**规则**'标签下的'**页面规则**'，为```blog.flartiny.cloudns.ch/*```添加一条'**始终使用https**'规则
3. 在wordpress设置中将**siteurl和home**都改成```https://blog.flartiny.cloudns.ch```

补充说一下这些操作主要是为了成功以我们的域名来访问博客网站，我个人没执行好这些步骤时遇到了意料之外的错误，比如：

1. 完全不能通过域名访问
2. 能访问博客主页，但不能访问wordpress管理面板
3. 能访问管理面板，但提示权限不足之类的

归结来说推测是陷入重定向循环之类的问题，也没太找到好的解决方法，于是自己摸索出一个还算可行的方案，不知道后面会不会有问题

---
再提一句，我用的是海外的服务器所以免去了备案这一步，有需要的可以自行操作
最后的最后附上一些其他**工具 / 解决方案**

1. 最关键的教程，部署wordpress几乎是按照这个来的 https://www.youtube.com/watch?v=pOESHd1G-HI&list=LL&index=1
2. 测试SSL认证 https://www.liquidweb.com/tools/ssl-checker/
3. 上传的文件尺寸超过upload_max_filesize https://www.cnblogs.com/live41/p/14736986.html
4. 修改siteurl和home后无法重新登录访问 https://jhchen.top/wordpress/wordpress-broken-after-changing-the-url/
5. 主题美化，我选择了Sakurairo，参考 https://github.com/mirai-mamori/Sakurairo
