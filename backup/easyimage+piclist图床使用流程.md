docker及docker-compose不再赘述，以下命令中只需关注8081的端口部分，可根据占用情况自行调整
```
docker run -itd \
  --name easyimage \
  -p 8081:80 \
  -e TZ=Asia/Shanghai \
  -e PUID=1000 \
  -e PGID=1000 \
  -v /var/easyimage/config:/app/web/config \
  -v /var/easyimage/i:/app/web/i \
  ddsderek/easyimage:latest
```

```
sudo apt update
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
// 编辑nginx配置文件
sudo vim /etc/nginx/sites-available/eimg
```
写入以下内容，其中eimg.example.com可以替换为你的域名，之前docker命令时设置映射到8081端口，请根据实际情况修改
```
server {
    listen 80;
    server_name eimg.example.com;

    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

```
// 软链接nginx配置文件到sites-enabled目录
sudo ln -s /etc/nginx/sites-available/eimg /etc/nginx/sites-enabled/
// 检查配置文件是否有语法错误
sudo nginx -t
// 重启nginx
sudo systemctl restart nginx
```
现在可以尝试访问http://eimg.example.com/

首次登录会出现管理员账户的注册配置页面，以及可以配置域名相关信息，也可以之后在设置中配置
![register](https://eimg.flartiny.cloudns.ch/i/2024/11/28/x4lsbv-0.png)

---
要配合PicList使用需要首先在easyimage设置-图床安全中开启API上传，然后转到API设置获取API地址以及token
填入PicList的图床设置-高级自定义图床中，配置格式如下
![setting](https://eimg.flartiny.cloudns.ch/i/2024/11/28/x9w77u-0.png)

---
缺点就是PicList的云端删除功能似乎不支持