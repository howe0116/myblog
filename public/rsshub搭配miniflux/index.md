# RSShub搭配Miniflux



> 😏简单记录一下步骤，在Centos8.2使用Docker&docker-compose

## RSS

Really Simple Syndication（简易信息聚合），它是一种消息来源的格式规范，网站可以按照这种格式规范提供文章的标题、摘要、全文等信息给订阅用户，用户可以通过订阅不同网站 RSS 链接的方式将不同的信息源进行聚合，在一个工具里阅读这些内容。

## Miniflux

使用 Docker Compose 安装 Miniflux:

- 创建 ~/miniflux 目录
`mkdir ~/miniflux && cd ~/miniflux`

- 创建 docker-compose.yml
  `touch docker-compose.yml`

- 修改 docker-compose.yml

  ```yaml
   version: '3'
  
  services:
  
    miniflux:
      image: miniflux/miniflux:latest
      ports:
        - "8080:8080"
      depends_on:
        - db
      environment:
        - ADMIN_USERNAME=admin
        - ADMIN_PASSWORD=password
        - BASE_URL=域名
        - CREATE_ADMIN=1
        - DATABASE_URL=postgres://miniflux:secret@db/miniflux?sslmode=disable
        - RUN_MIGRATIONS=1
  
    db:
      image: postgres:latest
      environment:
        - POSTGRES_USER=miniflux
        - POSTGRES_PASSWORD=secret
      volumes:
        - miniflux-db:/var/lib/postgresql/data
  
  volumes:
    miniflux-db:
   
  ```
  
- 运行 `docker compose up -d`

- 输入 `IP:<端口>` 应该已经可以访问 Miniflux了,接着给Miniflux弄个域名

## 配置 HTTPS

之前搭建halo时用的oneinstack配合nginx，所以依样画葫芦将Rsshub和miniFlux弄好域名

- 使用oneinstack创建站点

- 命令:

``` bash
cd oneinstack
sh vhost.sh
```

- 按照提示选择或输入相关信息即可

注:Miniflux地址配置完得去`docker-compose.yml`把ip地址改成域名。
### 使用Miniflux
- 设置完域名，就可以域名访问Miniflux了~

- 登录后可以通过设置修改语言，时区和主题等，还能设置PWA模式(渐进式网页应用显示模式)，在PC端使用效果不错。
![PWA](https://pic.howe0116.com/img/20220430202546.png)
- 于`设置-集成`处启用feverAPI并设置账号密码，以配合[Fluent 阅读器](https://github.com/yang991178/fluent-reader-lite/releases)(Android)、[FeedMe](https://play.google.com/store/apps/details?id=com.seazon.feedme)(Android)、[Reeder 5](https://reederapp.com/)(IOS)等阅读器在移动端使用。

## 用Rsshub获取一些订阅源

其实当下的网络RSS 这一信息获取方式早已式微，许多内容提供者（如`微信公众号`,`微博`等）并不会提供RSS，不过可以通过 RSSHub 项目获取一些订阅源。
> [RSShub](https://docs.rsshub.app/)是一个开源、简单易用、易于扩展的 RSS 生成器，可以给任何奇奇怪怪的内容生成 RSS 订阅源。RSSHub 借助于开源社区的力量快速发展中，目前已适配数百家网站的上千项内容。
> 可以配合浏览器扩展 [RSSHub Radar](https://github.com/DIYgod/RSSHub-Radar) 和 移动端辅助 App [RSSBud](https://github.com/Cay-Zhang/RSSBud)(iOS) 与 [RSSAid](https://github.com/LeetaoGoooo/RSSAid)(Android) 食用

### 使用Docker部署

1. 安装 `docker pull diygod/rsshub`

2. 运行

``` bash
 docker run -d --name rsshub -p 1200:1200 diygod/rsshub
```

3.访问 <http://ip:1200>即可，更详细的[看文档](https://docs.rsshub.app/)，然后也配个域名



## flowerss：通过TelegramBot订阅 



> [flowerss bot](https://flowerss-bot.vercel.app/#/?id=flowerss-bot)是一个支持应用内即时预览的 Telegram RSS Bot。

### Docker 部署

1.下载配置文件 在项目目录下新建 `config.yml` 文件

```bash
mkdir ~/flowerss &&\
wget -O ~/flowerss/config.yml \
    https://raw.githubusercontent.com/indes/flowerss-bot/master/config.yml.sampleCopy to clipboardErrorCopied
```

2.修改配置文件`config.yml`

- 主要是配置项` bot_token : Telegram上用BotFather创建机器人并拿到Token；`

- 遇到的困难：国内服务器访问不了api.telegram.org

​		解决方案：1. sock5代理(我没有)，2. CloudFlare Worker 反代,自定义telegram bot api url;

- 填写配置项：telegram.endpoint


修改配置文件中sqlite路径（如果使用sqlite作为数据库）：

```yaml
sqlite:
  path: /root/.flowerss/data.db
```

3.运行

```shell
docker run -d -v ~/flowerss:/root/.flowerss indes/flowerss-bot
```



## 使用感受

本来预定的计划是TelegramBot用来获取一些资讯，Miniflux则是用来订阅一些博客文章，

使用了三天的感受：

Miniflux那边就是半天没更新一章，基本每次打开都是0未读；Bot那边则刷刷刷很多资讯。


