# Real-time-Synchronization

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

