---
title: ubuntu下用hexo搭建静态博客
tags: [nodejs]
---
hexo是基于nodejs开发的一款静态博客的框架，搭建时比较麻烦，如果需要放到网上，则需要配合github一起使用。因此搭建博客环境，需要用到nodejs、hexo、git，同时博文是使用
markdown语法来写的，所以需要对markdown语法有一定的了解。

<!--more--> 
## 安装git

1、apt-get直接在线安装
``` bash
   $ sudo apt-get install git
```
   检查下是否安装成功：
``` bash
   git --version
```

2、配置用户名及email，否则后面在部署到github上时会报错
``` bash
   git config --global user.name "XXXXX"
   git config --global user.email "XXXXX"
```

## 安装nodejs

1、卸载ubuntu自带的nodejs和npm
   我的ubuntu版本是16.04LTS，本身自带的有nodejs和npm，自带的nodejs版本过低，且路径过于分散，不利用管理，所以我决定手动安装nodejs
``` bash
   sudo apt-get remove nodejs && sudo apt-get remove npm
```
   清理下依赖包
``` bash
   sudo apt-get autoremove
```

2、官网下载nodejs
``` bash
   cd /opt && wget https://nodejs.org/dist/v4.4.5/node-v4.4.5-linux-x64.tar.xz
   xz -d node-v4.4.5-linux-x64.tar.xz && tar vxf node-v4.4.5-linux-x64.tar
```

3、配置环境变量
``` bash
    sudo vi /etc/profile
```
   文件最后加上
   export PATH=$PATH:/opt/node-v4.4.5-linux-x64/bin
   同步文件使环境变量生效
``` bash
   source /etc/profile
```
4、测试安装
``` bash
   node --version && npm --version
```

## 安装hexo
1、在线安装cnpm
   由于npm使用国外站点下载hexo速度太慢，经常网络错误而使下载终断，所以我决定用cnpm在淘宝镜像上下载hexo
``` bash
   npm install -g cnpm --registry=https://registry.npm.taobao.org
   cnpm --version
```

2、在线安装hexo-cli
``` bash
   cnpm install -g hexo-cli
```
   经过一长串刷屏后就done了，检查是否安装成功
``` bash
   hexo --version
```

## 构建静态blog
1、初始化blog本地路径
``` bash
   cd /home/lc/workspace/blog && hexo init
```

2、生成node_modules
``` bash
   npm install
```

3、生成本地博客并测试
``` bash
   hexo g
   hexo server
```
   在浏览器输入http://localhost:4000，如果出现hexo标准页面，恭喜你，本地博客已经搭建完成

## 部署到github
1、申请github帐户，www.github.com，这里就不罗嗦了，不会英文可以百度翻译。

2、新建一个Repository，仓库名称为"your_github_username.github.io"，固定使用这个写法，不然会404

3、修改/home/lc/workspace/blog下的_config.yml文件，关联2中新建github仓库
   deploy:
     type: git
     repository: git@github.com:your_github_username/your_github_username.github.io.git
     branch: master
   其中repository有两种方式，一种是https的方式，一种是ssh的方式，由于我使用https方式时，使用用户名密码进行提交时鉴权各种通不过，所以我使用ssh方式提交代码。

4、生成ssh鉴权公私钥
``` bash
   $ ssh-keygen -t rsa -C "emailaddress@example.com"
```
   一路回车后，会在用户目录下生成.ssh的目录，里面有两个文件id_rsa，这是私钥，保存好；别一个是id_rsa.pub，这是公钥，需要配置到github上
5、github配置公钥
   在githhub上setting>SSH and GPG keys>new SSH key，title随便写，能方便区别不同key就行，将id_rsa.pub文件中的内容复制到key中。
6、发布到github上
``` bash
   hexo g && hexo d
```
   在浏览器中输入地址“your_github_username.github.io”，熟悉的页面是不是出现了。

## 绑定域名
1、购买域名并增加域名解析
   我是在阿里云上买的域名，买域名的时候需要注意买国际域名，否则需要审批备案。在阿里云中对该域名新增解析，地址指向your_github_username.github.io。

2、hexo绑定域名
   在/home/lc/workspace/blog/public下新建一个文件CNAME，将购买的域名写入该文件，注意格式xxxxx.com，不需要www。
``` bash
   $ cd /home/lc/workspace/blog && hexo g && hexl d
```
   将CNAME push到github上，这样就可以用自己的域名访问博客了。

## 更换主题
hexo有很多开源主题可以使用，其中比较流行的是hexo-next。
1、下载hexo-theme-next到本地博客路径
``` bash
   $ cd /home/lc/workspace/blog && git clone https://github.com/iissnan/hexo-theme-next
```
   下载完后在theme下就会看到多了一个next的目录。

2、配置主题
   在博客的配置文件_config.yml中，配置如下信息：
   theme: next
``` bash
   $ cd /home/lc/workspace/blog && hexo g && hexl d
```

## 其它配置
   博客配置文件_config.yml中的一些配置
1、页面中文
   language: zh-Hans
2、主页头
   title: RoyceLee's blog
3、描述，这个可以方便搜索引擎找到你的页面
   description: ×××××××××××××××××

## 发布新博客
1、写博文
   如果很熟悉md语法的话，那就直接vi或者geditd直接写，如果不熟悉并且博客内容有一些图片等富文本内容时，则推荐一个md在线编辑器[cmd markdown][1]
   在/home/lc/workspace/blog/source/的_posts目录中，新建一个md文件，可以直接编辑这个文件用md语法写博客，也可以在cmd markdown中编辑好复制进去。

2、发布
``` bash
   $ cd ../../ && hexo g && hexo d
```

## 标签页面
1、新建一个页面
``` bash
   $ hexo new page "tags"
```
2、编辑页面
   在source/tags/下配置index.md文件如下：
``` markdown
   ---
   title: tags
   date: 2016-06-14 10:28:42
   type: "tags"
   ---
```
3、修改主题配置文件
   在主题的_config.yml文件中如下配置：
``` markdown
  menu:
  home: /
  #categories: /categories
  #about: /about
  archives: /archives
  # tags菜单指向新建的面页
  tags: /tags
  #commonweal: /404.html
```



[1]: https://www.zybuluo.com/mdeditor
