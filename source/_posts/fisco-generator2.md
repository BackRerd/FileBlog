---
title: fisco-generator2
date: 2025-03-15 19:12:15
tags: ['fisco']
---

# 前置条件:

| 机构  |   节点   | 所属群组 | P2P地址 | Channel监听地址 | json监听地址 | IP地址    |
| ----- | :------: | -------- | ------- | --------------- | ------------ | --------- |
| 机构A |  节点0   | 群组1,2  | 30300   | 20200           | 8545         | 127.0.0.1 |
|       |  节点1   | 群组1,2  | 30301   | 20201           | 8546         | 127.0.0.1 |
| 机构B | 节点0(2) | 群组1    | 30302   | 20202           | 8547         | 127.0.0.1 |
|       | 节点1(3) | 群组1    | 30303   | 20203           | 8548         | 127.0.0.1 |
| 机构C | 节点0(4) | 群组2    | 30304   | 20204           | 8549         | 127.0.0.1 |

下载工具Generator

```sh
git clone https://gitee.com/FISCO-BCOS/generator.git #使用这个
```

切进目录并进行安装

```sh
cd generator/
./scripts/install.sh
# 测试是否安装成功
./generator -h 
```

![image.png](https://s2.loli.net/2025/01/14/URS4lqVJjimCO8E.png)

下载并解压二进制文件

```sh
git clone https://gitee.com/BackRe/resource.git
tar -zxvf ./resource/fisco-bcos.tar.gz -C ./meta
```

查看fisco版本信息

```sh
./meta/fisco-bcos -v
```

# 准备工作

根据前置条件中可以看到需求是ABCD机构所以要将当前的文件复制成ABCD四份。

```sh
cp -r generator/ generator-a
cp -r generator/ generator-b
```

![image.png](https://s2.loli.net/2025/01/14/8zHqkDAj1rblKg6.png)

生成区块链的唯一密钥证书(在generator目录下生成)

```sh
./generator --generate_chain_certificate ./chain_ca
```

`--generate_chain_certificate`生成区块链唯一的密钥相当于之前搭建中nodes/127.0.0.1/sdk内的内容

`./chain_ca`生成的文件名称

```sh
# 预输出效果如下
root@backrer-virtual-machine:~/fisco/generator# ./generator --generate_chain_certificate ./dir_chain_ca
INFO |  Chain cert begin.
INFO |  Generate root cert success, dir is /root/fisco/generator/dir_chain_ca
INFO |  Chain cert end.
```

然后生成各个机构的证书及密钥,并复制到指定的机构目录下的meta中

```sh
./generator --generate_agency_certificate ./agency_ca ./chain_ca agencya
./generator --generate_agency_certificate ./dgency_ca ./chain_ca agencyb
cp -r ./agency_ca/agencya/* ../generator-a/meta/
cp -r ./agency_ca/agencyb/* ../generator-b/meta/
```

`--generate_agency_certificate`生成机构的密钥

`./agency_ca`生成的文件位置

`./chain_ca`区块链的唯一密钥

`agencyxx`机构名称

# 机构A,B的准备工作

## 准备

切换到generatora目录中，编辑conf/node_deployment.ini

根据提供的ip端口进行对应的修改

```sh
[group]
group_id=1

[node0]
; Host IP for the communication among peers.
; Please use your ssh login IP.
p2p_ip=127.0.0.1
; Listening IP for the communication between SDK clients.
; This IP is the same as p2p_ip for the physical host.
; But for virtual host e.g., VPS servers, it is usually different from p2p_ip.
; You can check accessible addresses of your network card.
; Please see https://tecadmin.net/check-ip-address-ubuntu-18-04-desktop/
; for more instructions.
rpc_ip=127.0.0.1
channel_ip=0.0.0.0
p2p_listen_port=30300
channel_listen_port=20200
jsonrpc_listen_port=8545

[node1]
p2p_ip=127.0.0.1
rpc_ip=127.0.0.1
channel_ip=0.0.0.0
p2p_listen_port=30301
channel_listen_port=20201
jsonrpc_listen_port=8546
```

同理切换到generatorb,编辑conf/node_deployment.ini

```sh
[group]
group_id=1

[node0]
; Host IP for the communication among peers.
; Please use your ssh login IP.
p2p_ip=127.0.0.1
; Listening IP for the communication between SDK clients.
; This IP is the same as p2p_ip for the physical host.
; But for virtual host e.g., VPS servers, it is usually different from p2p_ip.
; You can check accessible addresses of your network card.
; Please see https://tecadmin.net/check-ip-address-ubuntu-18-04-desktop/
; for more instructions.
rpc_ip=127.0.0.1
channel_ip=0.0.0.0
p2p_listen_port=30302
channel_listen_port=20202
jsonrpc_listen_port=8547

[node1]
p2p_ip=127.0.0.1
rpc_ip=127.0.0.1
channel_ip=0.0.0.0
p2p_listen_port=30303
channel_listen_port=20203
jsonrpc_listen_port=8548
```

生成机构A的节点证书和p2p连接信息文件(切换到generator-a)

```sh
./generator --generate_all_certificates ./agencya_node_info
ll agencya_node_info
```

将p2p连接信息文件复制到agency-b中

```sh
cp agencya_node_info/peers.txt  ../generator-b/meta/peersa.txt
```

同理生成机构B的节点证书和p2p连接信息文件(切换到generator-b)

```sh
./generator --generate_all_certificates ./agencyb_node_info
```

将p2p连接信息文件复制到agency-a中

```sh
cp agencyb_node_info/peers.txt  ../generator-a/meta/peersb.txt
```

这里要注意一点，由于目前是要生成群组1需要创世区块，我们要将机构机构B中的两个cert文件拷贝到机构A的meta中。

```sh
cp -r agencyb_node_info/cert_*  ../generator-a/meta/
```

切换到机构A中并编辑文件`conf/group_genesis.ini`

```sh
cd ../generator-a/
vim conf/group_genesis.ini
```

```sh
[group]
group_id=1

[nodes]
node0=127.0.0.1:30300
node1=127.0.0.1:30301
node2=127.0.0.1:30302
node3=127.0.0.1:30303
```

`group_id`群组号

根据前置要求表中配置好node的ip以及端口

结合四个节点证书生成创世块.

```sh
./generator --create_group_genesis ./group
```

```sh
root@backrer-virtual-machine:~/fisco/generator-a# ./generator --create_group_genesis ./group
INFO |  Build operation begin.
INFO | generate ./group/group.1.genesis, successful
INFO |  Build operation end.
```

![image.png](https://s2.loli.net/2025/01/14/DYLCIBvdsoxkSO9.png)

将`group.1.genesis`文件复制到机构B中

```sh
cp group/group.1.genesis  ../generator-b/meta/
```

## 生成机构A

在generator-a文件下

使用`--build_install_package`生成机构的所有节点。

```sh
./generator --build_install_package ./meta/peersb.txt ./nodea
ll ./nodea
```

`./meta/peersB.txt`放置群组内其他节点的连接信息

`./nodeA`节点文件的输出目录

启动机构A的两个节点

```sh
bash nodea/start_all.sh
# 查询启动状态
ps -ef | grep fisco
```

![image.png](https://s2.loli.net/2025/01/14/INfzacL68JdSmHW.png)

## 生成机构B

切换到机构B

```sh
cd ../generator-b
```

生成机构B,并且启动

```sh
./generator --build_install_package ./meta/peersa.txt ./nodeb
bash nodeb/start_all.sh
# 查询启动状态
ps -ef | grep fisco
```

查看群组1的共识情况

```sh
tail -f ./node*/node*/log/log*  | grep +++
```

