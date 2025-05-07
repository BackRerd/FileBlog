---
title: 运维工程师
date: 2025-03-16 11:49:27
tags: ['定稿']
---

使用Fisco+Webase工具，完成多机构联盟链网络搭建，部署Webase可视化平台，确保节点跨群组互通与智能合约部署环境就绪  

| 机构  |   节点   | 所属群组 | P2P地址 | Channel监听地址 | json监听地址 | IP地址    |
| ----- | :------: | -------- | ------- | --------------- | ------------ | :-------- |
| 机构A |  节点0   | 群组1,2  | 30300   | 20200           | 8545         | 127.0.0.1 |
|       |  节点1   | 群组1,2  | 30301   | 20201           | 8546         | 127.0.0.1 |
| 机构B | 节点0(2) | 群组1    | 30302   | 20202           | 8547         | 127.0.0.1 |
|       | 节点1(3) | 群组1    | 30303   | 20203           | 8548         | 127.0.0.1 |
| 机构C | 节点0(4) | 群组2    | 30304   | 20204           | 8549         | 127.0.0.1 |

根据项目经理下发的任务书，我首先来完成区块链网络的搭建，为接下来整个项目的分布式账本以及用户管理链做好准备。

```sh
git clone https://gitee.com/FISCO-BCOS/generator.git
cd ./generator
```

```sh
./generator -h
```

这里由于项目的兼容性，我首先要获取对应版本的二进制文件，确定好区块链网络的版本。

```sh
git clone https://gitee.com/BackRe/resource.git
```

```sh
tar -zxvf ./resource/fisco-bcos.tar.gz -C ./meta
```
获取之后查询一下版本信息，这里使用的Fisco2.x的版本，这个版本对项目的兼容性较好。
```sh
./meta/fisco-bcos -v
```

对于我们初期的项目，暂时使用两个机构即可。

```sh
cd ..
cp -r ./generator a
cp -r ./generator b
cd ./generator
```

由于安全性，我们的整体项目需要一条唯一的一串密钥。

```sh
./generator --generate_chain_certificate ./chain_ca
```

然后根据我们的唯一密钥，去分发各个机构的密钥。

```sh
./generator --generate_agency_certificate ./agency_ca ./chain_ca agencya
./generator --generate_agency_certificate ./agency_ca ./chain_ca agencyb
```

```sh
cp -r ./agency_ca/agencya/* ../a/meta/
cp -r ./agency_ca/agencyb/* ../b/meta/
```

我们的整体项目需要与其他语言进行对接，所有我要提前准备好SDK连接密钥，便于其他语言的对接。

```sh
./generator --generate_sdk_certificate ./sdk_ca_a ./agency_ca/agencya/
./generator --generate_sdk_certificate ./sdk_ca_b ./agency_ca/agencyb/
```

```sh
cp -r sdk_ca_a/ ../a/
cp -r sdk_ca_b/ ../b/
```

以上我生成好了整个区块链中的所有密钥，现在我要开始进行区块链的初始化文件配置。

```sh
cd ../a
vim conf/node_deployment.ini
```

```sh
# 修改以下信息
p2p_listen_port=30300
channel_listen_port=20200
jsonrpc_listen_port=8545
```
我们项目对端口没有硬性的要求，所以在这里我只需要对端口进行自增一位数即可。
```sh
cd ../b
vim conf/node_deployment.ini
```

```sh
# 修改以下信息
p2p_listen_port=30302
channel_listen_port=20202
jsonrpc_listen_port=8547

p2p_listen_port=30303
channel_listen_port=20203
jsonrpc_listen_port=8548
```

初始化配置完成后，我需要将两个机构进行连接，否则无法实现分布式分发，所以我现在将两个机构的连接信息文件创建出来。

```sh
cd ../a
./generator --generate_all_certificates ./node_info
```

```sh
cd ../b
cp ../a/node_info/peers.txt ./meta/peersa.txt
```

```sh
./generator --generate_all_certificates ./node_info
cp ./node_info/peers.txt ../a/meta/peersb.txt
```

```sh
cp ./node_info/cert_* ../a/meta/
```

由于我们项目采取的是多群组模式，所以现在需要将对应的群组信息初始化出来。

```sh
cd ../a
vim ./conf/group_genesis.ini
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

目前为止区块链的初始化和密钥已经生成创建完成。

由于区块链是以链条的形式存在，每一个块都会继承上一个块的信息，但目前我的区块链网络是空的，所以我需要创建出第一个区块，并分发给各个机构。

```sh
./generator --create_group_genesis ./group
```

看到`generate ./group/group.1.genesis, successful`信息表示生成成功。

```sh
cp -r ./group/group.1.genesis ../b/meta/
```

第一个区块存在后，我将开始着手构建机构A链。

```sh
./generator --build_install_package ./meta/peersb.txt ./nodea
```

看到`Build operation end`表示机构A生成完毕。

启动机构a的所有节点,并查询启动状态

```sh
bash ./nodea/start_all.sh
ps -ef | grep fisco
```

看到以下有节点信息，代表启动正常。

![image.png](https://s2.loli.net/2025/03/16/ZEheYOJPksFplvH.png)

同理生成机构b链,并启动查看。

```sh
cd ../b
./generator --build_install_package ./meta/peersa.txt ./nodeb
bash ./nodeb/start_all.sh
ps -ef | grep fisco
```

看到以下节点信息，代表所有节点已经启动成功。以上我已完成了整个项目的区块链搭建。

![image.png](https://s2.loli.net/2025/03/16/DF5R6fXMKVdhygA.png)

由于智能合约编写时需要可视化界面，以及更好的数据监控，我将部署Webase平台。

```sh
cd
git clone https://gitee.com/BackRe/webase.git #现在群里下
```

```sh
cd ./webase
```

```sh
python3 deploy.py installAll
```

```sh
python3 deploy.py startAll
```

到此，我的工作以完成，我将查询我的IP以便于其他人的连接。

```sh
ip addr
```

![image.png](https://s2.loli.net/2025/03/16/pjxWRJ2Fcesiar3.png)

(在ens33下，其中inet后面的地址就是ip地址。没有/24。最后的ip是:你的ip:5000)

报告项目经理按你下达的任务指令我以完成部署工作。

我将我的区块链ip等信息交给项目经理。

