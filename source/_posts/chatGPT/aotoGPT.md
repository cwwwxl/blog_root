---
title: AgentGPT 最新部署教程！
cover: 'https://s2.loli.net/2023/04/28/LI6U75vYfDSMq9n.png'
tags: AgentGPT
categories: AgentGPT
top_img: 'https://s2.loli.net/2023/04/28/LI6U75vYfDSMq9n.png'
abbrlink: 1f42c3fa
---

# AgentGPT 最新部署教程！ 

 
[![img](https://s2.loli.net/2023/05/18/EsreW8UmSwjMP4f.png)](https://www.freedidi.com/wp-content/uploads/2023/04/2023-04-26-224232.png)

#### 1.首先你需要准备一台VPS，没有的话可以自己去【**[搞一台](https://www.vultr.com/?ref=7045490)**】，白菜价，性能强劲而且可玩性非常高！

 然后通过SSH连接工具【[Putty](https://www.putty.org/)】连接进去以后，依次执行以下命令：

 如果没有安装Curl的话，我们需要提前安装。

```none
sudo apt install -y curl #Debian
```

```none
 sudo yum install -y curl #CentOS
```

#### 2.通过Docker一键安装脚本进行部署：

```none
bash <(curl -sSL https://cdn.jsdelivr.net/gh/SuperManito/LinuxMirrors@main/DockerInstallation.sh)
```



里面选择DOCKER CE源渠道，有八个可以选择。

包括我们可以选择Docker Engine、Compose等，然后就自动一键安装。

#### 3.下载并安装AgentGPT开源程序： 【**[GitHub项目](https://github.com/reworkd/AgentGPT)**】

先安装下Git：

```none
sudo yum install git #Centos
```

```none
sudo apt install git #Debian
```



#### 4.下载 AgentGPT 源码：

```none
git clone https://github.com/reworkd/AgentGPT.git
```



#### 5.下载后执行下面安装命令即可:

```none
cd AgentGPT

./setup.sh --docker
```



#### 6.获取自己的Open AI 的密钥：【**[点击前往](https://platform.openai.com/account/api-keys)**】

 

#### 7.安装完成以后，你只需在浏览器上打开：http:xxx.xx.xx.xx:3000