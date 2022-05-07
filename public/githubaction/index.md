# 使用 GitHub Action 推送博客到服务器

为了省去每次更新网站都要将hugo生成的public静态网页文件夹用XFTP传输到服务器这步骤，使用了Github Action自动推送到服务器上，以下是我的步骤:

1. 准备ssh密钥对，以验证登录服务器;
2. 新建一个github仓库,用来推Hugo生成的 public 文件夹;
3. 创建GitHubAction服务;

## 稍微了解一下GitHubAction

[GitHub Actions入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

## 生成SSH密钥对

想把文件部署到远程服务器，先要解决登录问题。可以用密码登录也可以用SSH密钥，我用的是SSH密钥;

1. 生成 SSH 密钥对：

```shell
$ mkdir -p ~/.ssh && cd ~/.ssh 
$ ssh-keygen -t rsa -f myblog 
Generating public/private rsa key pair. Enter passphrase (empty for no passphrase): 
Enter same passphrase again:
```

一直enter就行，会在~/.ssh生成`myblog`（私钥）和`myblog.pub`（公钥），私钥注意要保管好；

2. 将公钥`myblog.pub`的内容复制到~/.ssh/authorized_keys，或者直接

```shell
$ cat myblog.pub >> authorized_keys
```

3. 复制私钥内容备用；

## 创建Actions

登录GitHub创建一个仓库，推送public到仓库，并创建actions

![image-20220507151542414](https://pic.howe0116.com/img/image-20220507151542414.png)

### 添加secret

打开代码仓库-> `Settings`-> `Secrets(Actions)`-> `New repository secret` 添加以下

![image-20220507153040032](https://pic.howe0116.com/img/image-20220507153040032.png)



### 配置文件

copy自某位网友，来源找不着了 。

```yaml
name: Deploy site files

on:
  push:
    branches:
      - main # 只在main上push触发部署
    paths-ignore: 
      - README.md
      - LICENSE
      - .gitignore

jobs:
  deploy:
    runs-on: ubuntu-latest # 使用ubuntu系统镜像运行自动化脚本

    steps: # 自动化步骤
      - uses: actions/checkout@v2 # 1.下载代码仓库

      - name: Deploy to Server # 2.rsync推文件
        uses: AEnterprise/rsync-deploy@v1.0 # 使用别人包装好的步骤镜像
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }} # 引用配置，SSH私钥
          ARGS: -avz --delete public/ # rsync参数，
          SERVER_PORT: '22' # SSH端口
          FOLDER: ./public # 要推送的文件夹，路径相对于代码仓库的根目录
          SERVER_IP: ${{ secrets.SSH_HOST }} # 引用配置，服务器的host名（IP）
          USERNAME: ${{ secrets.SSH_USERNAME }} # 引用配置，服务器登录名
          SERVER_DESTINATION: /home/public # 部署到目标文件夹

```

###  推送

这样，每次生成public文件夹推送到GitHub，就可以自动上传到服务器了；

![image-20220507153557964](https://pic.howe0116.com/img/image-20220507153557964.png)



补充：因为只需要public文件夹push到 GitHub，忽略掉其他hugo源文件，创建`.gitignore`并 填入除了public之外的文件夹名称即可。

然后发表文章或者修改网站时，hugo命令生成public静态网页，打开GitHub desktop进行push。



就是每push一次，腾讯云就发一次高危警告，麻了。🤣

