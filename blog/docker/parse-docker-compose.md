---
date: "2018-09-15"
title: "Docker Compose 搭建单机的 Parse 服务"
category: "Docker"
---

## 一个文件就启动一台后端服务的可行性

自从入坑 Docker 以来，就越发觉得，如果客户端程序员，稍微能抽出一点空闲时间，学习一下新时代的服务端的构建技术，完全有希望在尚未掌握服务端运维技术的时候，自己搞一个小型的服务端。

这次要分享的就是沉迷已久的 [Parse Server][1] 的 Docker Compose 之旅。


## 从 npm start 开始

首先按照文档，在任意一台机器上（这一步建议在本地的 MacOS 上实现，什么？你用 Windows？祝你好运。）

```sh
$ npm install -g parse-server mongodb-runner
$ mongodb-runner start
$ parse-server --appId APPLICATION_ID --masterKey MASTER_KEY --databaseURI mongodb://localhost/test
```

搞完之后，你在本地就拥有了一台 [Parse Server][1] 的实例，你可以增删改查数据了:


```sh
curl -X POST \
-H "X-Parse-Application-Id: APPLICATION_ID" \
-H "Content-Type: application/json" \
-d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}' \
http://localhost:1337/parse/classes/GameScore
```

然鹅（大误），这不是我们想要的，这种运行环境是没有灵魂的，哪怕你在服务端的 Linux 机器上起来也完全没有新时代的意义，因为我们要讲的是「后端服务」，而现在充其量就是一个「后端程序」。

因为它有如下几个问题：

### 没有靠谱的前置 web 调度服务(apache/nginx)

当然你可以继续在本地安装一个 nginx 然后编写 conf 文件来转发，恕我直言，这一步已经需要一个完全没接触过后端的同学，学习至少 2 个小时。


### 没有一个 dashboard 来管理和查看数据

解决它也很简单，在本地继续部署一个 [Parse Dashboard][2]，问题是你本地就有至少要暴露 2 个端口在公网上，第一个是 [Parse Server][1] 的 1337 和 [Parse Dashboard][2] 的 4040，你要继续在你的云主机供应商的界面上去调整这些端口，这也是一步需要理解和学习的，没说学习新东西不好，而是说我们的目标是尽可能的快和简单。

### Web 服务没有 HTTPS 的安全加持

当然如果你前面两项都能很好解决的话，那你应该已经学会了 nginx 的配置，接下来稍微学习一下 https 加密和 LetsEncrypt 的也不是问题，这一步其实在任何配置 web 服务器的过程中都无法省略，因为域名是你注册的，证书也应该由你来申请。不过假设我们能节省掉前面两步，不是更好？


### 从 Docker run 再次切入

很开心，官网也给出了 Docker 的解决方案：

```sh
$ docker build --tag parse-server .
$ docker run --name my-mongo -d mongo
$ docker run --name my-parse-server --link my-mongo:mongo -d parse-server --appId APPLICATION_ID --masterKey MASTER_KEY --databaseURI mongodb://mongo/test
```

可是，你是在逗我？

这不跟之前的 npm start 毫无区别么？？？

## 简单介绍一下 Docker Compose 和 Docker 

对于从未接触过服务端容器技术的客户端程序员，先可以理解一下这句话：

> 把服务端的数据接收/数据处理/数据返回当做一次任务的话，那么 Docker 就是一个任务池，当池子里面的任务满了，Docker 可以很好的被宿主机知晓，并且很容易动态的分配更多的资源去处理任务调度。

而我们知道，服务端里面有各种类型的任务池，客户端的一次请求，比如登录，Web 前端只负责接收数据，并且将数据转发给后面的 Passport 服务，Web 的任务就完成了，而 Passport 的任务才刚开始，它需要将数据包里面的 username 和 password 拿出来，检验一下是不是字符串，然后它去访问 db 服务，从 db 服务那里查询比对，再将结果通知给 web 服务，web 服务将结果数据返回给客户端，因此在真正的服务端有至少三个这样的大池子：

- Web
- Passport
- Database

因此我们需要一个池子管理员，它来负责给所有的任务池进行任务调度和资源分配。

Docker Compose 就是这个管理员。

> Docker Compose 通过一个配置文件来启动所有对应的服务，然后这些服务在逻辑上都处在同一个内部子网中，相互之间可以通过对应的端口进行互访。

那么我们可不可以使用一个配置文件来统一解决前面部署 [Parse Server][1] 的问题呢？

## Docker Compose 出场

首先我们看一下我们最终要编写出来的配置文件 `docker-compose.yml`:

```yml
version: '3'
services:
  mongo:
    image: 'mongo:latest'
    volumes:
      - './data/db:/data/db'
    ports:
      - 27017:27017
  parse:
    image: 'parseplatform/parse-server:latest'
    environment:
      - PARSE_SERVER_APPLICATION_ID=bas
      - PARSE_SERVER_MASTER_KEY=bas@master123!
      - PARSE_SERVER_DATABASE_URI=mongodb://mongo:27017/bas
      - PARSE_SERVER_MOUNT_PATH=/bas
    ports:
      - '1337:1337'
    links:
      - mongo
  parse-dashboard:
    image: 'parseplatform/parse-dashboard'
    ports:
      - '4040:4040'
    environment:
      - PARSE_DASHBOARD_ALLOW_INSECURE_HTTP=true
      - PARSE_DASHBOARD_SERVER_URL=http://localhost:1337/bas
      - PARSE_DASHBOARD_MASTER_KEY=bas@master123!
      - PARSE_DASHBOARD_APP_ID=bas
      - PARSE_DASHBOARD_APP_NAME=BAS
      - PARSE_DASHBOARD_USER_ID=wujun
      - PARSE_DASHBOARD_USER_PASSWORD=basp@ssword
  nginx:
    image: 'nginx:latest'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - './nginx:/etc/nginx/sites-enabled'
      - './nginx/default.conf:/etc/nginx/conf.d/default.conf'
      # letsencrypt will put generated pems under /etc/letsencrypt/live/yourdomain.com, copy them to nginx container. 
      # - '/etc/letsencrypt/live/yourdomain.com/fullchain.pem:/etc/nginx/ssl/fullchain.pem'
      # - '/etc/letsencrypt/live/yourdomain.com/privkey.pem:/etc/nginx/ssl/privkey.pem'
    links:
      - parse-dashboard
      - parse
```

你没看错，只要这一个文件，你在任何一台服务器上就可以启动一个较为科学的 [Parse Server][1] 。


根据这个配置文件，稍微知道一点 yml 格式文件的同学也大概看懂了一二。

接下来，可以在任意一台服务器（推荐 Ubuntu）上执行如下脚本：

```sh
$ git clone https://github.com/wujun4code/docker-compose-parse-server.git
$ cd docker-compose-parse-server
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
$ sudo docker-compose up -d
```

这样你的 [Parse Server][1] 就启动了：

你可以通过浏览器打开 : `http://你的服务器 ip` 就可以看见一个 [Parse Dashboard][2] 页面，

而且通过如下 `curl` 命令就可以存储一个对象:

```sh
curl -X POST \
-H "X-Parse-Application-Id: APPLICATION_ID" \
-H "Content-Type: application/json" \
-d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}' \
http://你的服务器 ip/bas/classes/GameScore
```

或者你可以直接在 [Parse Dashboard][2] 添加。


### HTTPS 安全加持

这一步需要费一点事儿，尽管 Docker 的社区已经有了被人魔改的 Image 可以自动化，但是我个人的坚持是尽量使用官网的 Image 来构建，因为你不确定第三方的什么时候就停更了，万一出问题了，你真的会被坑很久。

#### 前置条件

- 你拥有一个域名 - namesilo/狗爹都可以注册
- 服务器最好在海外（香港/日本/俄罗斯都可以）

操作步骤是先停掉正在运行的 nginx:

```s
# 在 docker-compose-parse-server 目录下
$ sudo docker-compose stop
$ sudo apt-get install letsencrypt
$ sudo letsencrypt certonly --standalone
```
根据提示输入你的注册域名时的邮箱和域名：

```sh
$ xxx@gmail.com
$ youdomain.com www.yourdomain.com
```


然后修改 docker-compose-parse-server/nginx/default.conf 成如下内容：

（其实就是把原本注释取消掉）

```conf
server {
     listen 80;
     listen [::]:80 default_server ipv6only=on;
     return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;


    ssl on;
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

    location /api/ {
         proxy_pass http://parse:1337/;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header X-NginX-Proxy true;
         proxy_ssl_session_reuse off;
         proxy_set_header Host $http_host;
         proxy_redirect off;
    }

    location / {
         proxy_pass http://parse-dashboard:4040/;
         proxy_set_header Host $host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header X-Forwarded-Proto https;
    }
}
```

并且还要修改一次 `docker-compose.yml`:

把原本注释掉的部分还原回来:

```yml
    volumes:
      - './nginx:/etc/nginx/sites-enabled'
      - './nginx/default.conf:/etc/nginx/conf.d/default.conf'
      # letsencrypt will put generated pems under /etc/letsencrypt/live/yourdomain.com, copy them to nginx container. 
      - '/etc/letsencrypt/live/yourdomain.com/fullchain.pem:/etc/nginx/ssl/fullchain.pem'
      - '/etc/letsencrypt/live/yourdomain.com/privkey.pem:/etc/nginx/ssl/privkey.pem'
```

然后重新启动:

```sh
$ sudo docker-compose up -d --force-recreate
```

一切搞定，你用 https 访问你的域名，此刻你就拥有一个 [Parse Server][1] 的单机实例了。

集群怎么做呢？

那是另一个话题： Docker Swarm

[1]: https://github.com/parse-community/parse-server
[2]: https://github.com/parse-community/parse-dashboard



