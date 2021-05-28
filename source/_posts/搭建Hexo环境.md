---
title: 云服务搭建Hexo
date: 2018-05-01
tags: [运维,Hexo]
---

### 安装配置Hexo

安装Nodejs

```
下载nodejs
wget https://nodejs.org/dist/v8.11.2/node-v8.11.2-linux-x64.tar.xz

解包
tar xf node-v8.11.2-linux-x64.tar.xz

建立软连接，此处源目录最好是绝对路径，否则可能会报错
ln -s 源目录/node /usr/local/bin/node
ln -s 源目录/npm /usr/local/bin/npm
ln -s 源目录/npx /usr/local/bin/npx
```

安装Hexo

```
npm install hexo-cli -g

建立软连接
ln -s 源目录/hexo /usr/local/bin/hexo
```

### 安装配置Nginx

阿里云CentOS7自带yum源安装

```
yum install nginx -y
```

启动服务

```
手动启动
systemctl start nginx

设置开机自启
systemctl enable nginx
```

开放阿里云80端口，访问测试

![](http://wx2.sinaimg.cn/large/be961c2aly1fs9yb2mhahj21cp0ggq3z.jpg)

修改配置文件，打开/etc/nginx/nginx.conf，修改root以配置博客默认文件夹

```
server {
    ...
    root         /usr/share/nginx/blog;
    ...
}
```

在/usr/share/nginx/下新建blog文件夹作为展示博客的根目录，重启Nginx



### hexo配置

创建hexo目录，在文件夹中初始化hexo

```
hexo init
```

修改hexo目录下_config.yml文件，修改public_dir使其指向Nginx下的博客目录，便可以直接生成到网站目录下

```
public_dir: /usr/share/nginx/blog
```

在hexo目录下生成博客，并访问测试

```
hexo g
```

![](http://wx2.sinaimg.cn/large/be961c2aly1fs9yb5asiij21fu0u0wv0.jpg)