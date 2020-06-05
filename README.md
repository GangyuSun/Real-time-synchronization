# Linux普通用户的文件实时同步增量备分

## 配置免密AB机器
- 源服务器 A 
- 同步服务器 B
#### 1. Linux下生成密钥（ssh-keygen—生成、管理和转换认证密钥）

查看源服务器home下".ssh"路径中是否存在公钥。
若不存在通过
```shell 
ssh-keygen -t rsa
```
生成公钥。\
查看home下.ssh文件夹：
```
* authorized_keys:存放远程免密登录的公钥,主要通过这个文件记录多台机器的公钥（初始不存在该文件） *
* id_rsa : 生成的私钥文件 *
* id_rsa.pub ： 生成的公钥文件*
* know_hosts : 已知的主机公钥清单*
```

#### 2. 把公钥拷贝到B机器对应用户家目录authorized_keys
```shell
command: ssh-copy-id -i ~/.ssh/id_rsa.pub [romte_ip]
example: ssh-copy-id -i ~/.ssh/id_rsa.pub user1@xx.xx.xx.xx
```
![](https://github.com/SunssAria/Real-time-Synchronization/blob/master/ssh-copy-id.png)
记住更改同步服务器authorized_keys的权限：`chmod 600 ~/.ssh/authorized_keys`

## 部署rsync
需要管理员权限,一般情况下服务器都会支持rsync

## 部署inotify
#### 1. 首先检查系统内核是否支持 inotify
```shell
uname -r  
ll /proc/sys/fs/inotify
```
出现以下说明及文件说明可以安装
```
[user@localhost ~]# uname -r  #查询系统内核版本
3.10.0-327.el7.x86_64
[user@localhost ~]# ll /proc/sys/fs/inotify
total 0
-rw-r--r-- 1 root root 0 Mar 11 09:34 max_queued_events
-rw-r--r-- 1 root root 0 Mar 11 09:34 max_user_instances
-rw-r--r-- 1 root root 0 Mar 11 09:34 max_user_watches
```
#### 2. 下载安装
下载地址：https://links.jianshu.com/go?to=http%3A%2F%2Fgithub.com%2Fdownloads%2Frvoicilas%2Finotify-tools%2Finotify-tools-3.14.tar.gz
```
#解压
#配置：
  ./configure --prefix={your path}
#安装编译：
  make && make install
```

#### 3. 编写同步脚本
vi [path]/inotifyrsync.sh
```shell
#!/bin/bash
host1=10.15.22.110
src=/home/data/Titan2/K2/xuwq2/
dst1=data/Cryo_data/
user1=xuell
/home/xuwq2/software/inotify-tools-3.14/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e close_write,modify,create,attrib $src \
| while read files
do
        /usr/bin/rsync -vzrt -e "ssh" --progress $src $user1@$host1:$dst1 > /home/xuwq2/process_manager/inotify_process/rsync.out 2>&1
        echo "${files} was rsynced." >> /home/xuwq2/process_manager/inotify_process/rsync.log 2>&1
done
```
其中 host1 是 client 的 ip，src 是 server 端要实时监控的目录，dst1 目的路径（不能以/开始），user1 是建立密码文件里的认证用户。\
然后给这个脚本赋予权限
`chmod 755 inotifyrsync.sh`
#### 4. 后台运行：
```shell
nohup ./inotifyrsync.sh >inotifyrsync.nohup 2>&1 &
```
