---
layout: post
title: "Linux 新建用户、删除用户"
date: 2022-09-02
tag: Linux
---

> 参考文章：
[Linux_创建新用户](https://zhuanlan.zhihu.com/p/413577022)
[linux创建新用户](https://blog.csdn.net/u9king/article/details/116261122)


📌 前言：最近白嫖了一个月的阿里云服务器，默认只有 root 用户，所以想新建一个普通用户使用。


# 1. 创建新用户 `abc`

在root账户下输入以下命令，创建 `abc` 用户。

```bash
useradd -m abc  #创建名为abc的新用户，并在/home目录下创建用户文件夹
passwd abc  #给新用户设置登录密码（输入密码时看不到的，直接输入即可）
usermod -s /bin/bash abc  #确保创建新用户也是使用的bash脚本
```

<img src="https://songhailong-1257323743.cos.ap-chengdu.myqcloud.com/Untitled.png" alt="Untitled" style="zoom:50%;" />

或者非root账号下，在每条命令前加 `sudo` ，每次执行时需输入root账号密码，如下

```bash
sudo useradd -m abc  #创建名为abc的新用户，并在/home目录下创建用户文件夹
sudo passwd abc  #给新用户设置登录密码（输入密码时看不到的，直接输入即可）
sudo usermod -s /bin/bash abc  #确保创建新用户也是使用的bash脚本
```

# 2. 可能出现的错误

登录新建的用户时，可能会提示以下错误：

```bash
/usr/bin/xauth:  file /home/abc/.Xauthority does not exist
```

- **错误原因：**
  
    添加用户时没有授权对应的目录，仅仅执行了useradd user而没有授权对应的家目录
    
- **解决办法：**
  
    使用 root 权限执行以下命令，改变指定abc用户目录及其内所有子文件的所属主与所属组
    
    ```bash
    chown abc:abc -R /home/abc
    ```
    
    <img src="https://songhailong-1257323743.cos.ap-chengdu.myqcloud.com/Untitled%201.png" alt="Untitled" style="zoom:80%;" />

再次登录 abc 用户，就不会再报错了。

> 参考：[Lunix下建立新用户-.Xauthority does not exist-不显示用户名和路径](http://t.csdn.cn/ychfQ)
> 

# 3. 给用户添加 sudo 权限（不需要可不设置）

使用 root 权限执行以下命令：

```bash
chmod u+w /etc/sudoers  #给sudoers文件添加可写的权限
vim sudoers  #使用vim进入编辑该文件，给新创的用户加上权限，保存退出
```

找到 `root` 用户行，按 `o` 键，在下方插入一行，输入

```bash
abc     ALL=(ALL:ALL) ALL
```

如下图所示

<img src="https://songhailong-1257323743.cos.ap-chengdu.myqcloud.com/Untitled%202.png" alt="Untitled" style="zoom:50%;" />

输入完成后，先按 `ESC` 键，再输入 `:wq` 后按 回车键，保存并退出。

<img src="https://songhailong-1257323743.cos.ap-chengdu.myqcloud.com/Untitled%203.png" alt="Untitled" style="zoom:80%;" />

再执行以下命令取消该文件的权限

```bash
chmod u-w sudoers  #再将该文件的权限关掉
```

至此，该用户拥有 `sudo` 权限。

# 4. 删除用户

在 root 权限下执行以下命令：

```bash
userdel -r abc  #在root权限下将测试用户删掉，并删除用户文件夹，然后退出测试用户的登陆即可
```

如下图所示

<img src="https://songhailong-1257323743.cos.ap-chengdu.myqcloud.com/uTools_1662125756649.png" alt="uTools_1662125756649.png" style="zoom:80%;" />

在通过以下指令查询该用户时，提示没有此用户

```bash
root@xxxxxx:~# id -u abc
id: ‘abc’: no such user
```

# 结尾

推荐一个linux教程：[http://c.biancheng.net/view/844.html](http://c.biancheng.net/view/844.html)