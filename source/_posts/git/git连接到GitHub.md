---
title: git连接到GitHub
cover: 'https://s2.loli.net/2023/01/22/cjp3dHrSYfst8BC.jpg'
tags: git
categories: git
top_img: 'https://s2.loli.net/2023/01/26/jFldL1tuzWbvpyq.jpg'
abbrlink: db6bedc5
---
## 1. git连接到github

### 1.1 安装git

```shell
sudo apt-get install git
```

### 1.2 创建一个GitHub账号

```shell
 https://github.com/
```

### 1.3 生成ssh key

```shell
ssh-keygen -t rsa -C "your_email@youremail.com"
```

- 一直点确定

![image-20230120105405140](https://s2.loli.net/2023/01/20/BCMZwDVQ4JjYmWG.png)

- 默认在C:\Users\Administrator\.ssh这个目录下
- ![image-20230120105639091](https://s2.loli.net/2023/01/20/pZWTBFqoeCKH9rR.png)

- 点击设置

![image-20230120105751375](https://s2.loli.net/2023/01/20/IT2JykZONbnlCzq.png)

![image-20230120105846939](https://s2.loli.net/2023/01/20/mUNdYM5ItGxJDO8.png)

![image-20230120105952246](https://s2.loli.net/2023/01/20/YrQjWCUVicIvkH3.png)

- 将密钥复制到GitHub里面

![image-20230120110124613](https://s2.loli.net/2023/01/20/LMVzlQU4itCONDx.png)

- 点击保存

### 1.4 配置git

```shell
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"
```

### 1.5 测试是否连接成功

```shell
ssh -T git@github.com
```

