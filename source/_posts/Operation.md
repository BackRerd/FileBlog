---
title: 运维工程师
date: 2025-03-16 11:49:27
tags: ['定稿']
---

所有人介绍项目以及口号，项目经理下发任务到运维工程师。已经向运维工程师下发了，任务书请查收。

| 机构  |   节点   | 所属群组 | P2P地址 | Channel监听地址 | json监听地址 | IP地址    |
| ----- | :------: | -------- | ------- | --------------- | ------------ | :-------- |
| 机构A |  节点0   | 群组1,2  | 30300   | 20200           | 8545         | 127.0.0.1 |
|       |  节点1   | 群组1,2  | 30301   | 20201           | 8546         | 127.0.0.1 |
| 机构B | 节点0(2) | 群组1    | 30302   | 20202           | 8547         | 127.0.0.1 |
|       | 节点1(3) | 群组1    | 30303   | 20203           | 8548         | 127.0.0.1 |
| 机构C | 节点0(4) | 群组2    | 30304   | 20204           | 8549         | 127.0.0.1 |

根据项目经理下发的任务书，我们来完成区块链部署的工作。

1.从我们的服务器上下载区块链搭建工具(generator):

```sh
git clone https://gitee.com/FISCO-BCOS/generator.git
cd ./generator
```

2.因为网络问题工具已经提前构建完成，所有我们先测试一下工具是否能使用。

```sh
./generator -h
```

3.根据上面显示的信息，可以知道工具可以正常使用。接下来我们需要完成区块链搭建的重要部分，获取区块链所使用的二进制文件。

```sh
git clone https://gitee.com/BackRe/resource.git
```

在获取完成后，需要将二进制文件解压到`./meta`目录下。meta目录相当于一个缓存文件，几乎所有的信息文件都会放在这里。

```sh
tar -zxvf ./resource/fisco-bcos.tar.gz -C ./meta
```

解压完成后，查询一下二进制文件的版本信息。

```sh
./meta/fisco-bcos -v
```

看到版本信息后代表文件正常解压成功。

4.我们需要将工具文件复制成a,b机构，为后续的智能合约做准备。

```sh
cd ..
cp -r ./generator a
cp -r ./generator b
cd ./generator
```

5.在复制完成之后，需要生成机构的唯一密钥和证书。

```sh
./generator --generate_chain_certificate ./chain_ca
```

看到`Generate root cert success,dir is /root/fisco/generator/dir_chain_ca`信息，机构的`唯一`密钥生成完毕,并且知道密钥生成在`/root/fisco/generator/dir_chain_ca`。

6.通过唯一密钥，我们需要生成各个机构的密钥。

```sh
./generator --generate_agency_certificate ./agency_ca ./chain_ca agencya
./generator --generate_agency_certificate ./agency_ca ./chain_ca agencyb
```

将当前生成的各个机构密钥，复制到各个机构目录下。

```sh
cp -r ./agency_ca/agencya/* ../a/meta/
cp -r ./agency_ca/agencyb/* ../b/meta/
```

生成sdk连接密钥证书

```sh
./generator --generate_sdk_certificate ./sdk_ca_a ./agency_ca/agencya/
./generator --generate_sdk_certificate ./sdk_ca_b ./agency_ca/agencyb/
```

同理,将当前生成的各个机构sdk，复制到各个机构目录下

```sh
cp -r sdk_ca_a/ ../a/
cp -r sdk_ca_b/ ../b/
```

7.接下来需要配置各个机构下的文件。

```sh
cd ../a
vim conf/node_deployment.ini
```

为机构a配置两个节点和对应的端口。

```sh
# 修改以下信息
p2p_listen_port=30300
channel_listen_port=20200
jsonrpc_listen_port=8545
```

b同理。

```sh
cd ../b
vim conf/node_deployment.ini
```

修改b的对应配置信息。

```sh
# 修改以下信息
p2p_listen_port=30302
channel_listen_port=20202
jsonrpc_listen_port=8547

p2p_listen_port=30303
channel_listen_port=20203
jsonrpc_listen_port=8548
```

8.生成机构A的节点信息和p2p连接信息

```sh
cd ../a
./generator --generate_all_certificates ./node_info
```

显示xxx信息表示节点信息和p2p连接信息生成成功。

将机构a `p2p`文件复制到机构b的`meta`目录下。

```sh
cd ../b
cp ../a/node_info/peers.txt ./meta/peersa.txt
```

同理将生成机构b的节点信息和p2p文件，然后再复制到机构a中。

```sh
./generator --generate_all_certificates ./node_info
cp ./node_info/peers.txt ../a/meta/peersb.txt
```

将机构b的节点信息复制到机构a的`meta`目录下。

```sh
cp ./node_info/cert_* ../a/meta/
```

切回机构a中，配置群组文件。

```sh
cd ../a
vim ./conf/group_genesis.ini
```

根据文档配置对应的节点信息。

```sh
[group]
group_id=1

[nodes]
node0=127.0.0.1:30300
node1=127.0.0.1:30301
node2=127.0.0.1:30302
node3=127.0.0.1:30303
```

生成区块链中的第一个区块。

```sh
./generator --create_group_genesis ./group
```

看到`generate ./group/group.1.genesis, successful`信息表示生成成功。

将第一个区块复制到机构b中

```sh
cp -r ./group/group.1.genesis ../b/meta/
```

9.现在生成机构a的节点。

```sh
./generator --build_install_package ./meta/peersb.txt ./nodea
```

看到`Build operation end`表示节点生成完毕。

启动机构a的所有节点,并查询启动状态

```sh
bash ./nodea/start_all.sh
ps -ef | grep fisco
```

看到以下有节点信息，代表启动正常。

![image.png](https://s2.loli.net/2025/03/16/ZEheYOJPksFplvH.png)

同理生成机构b的节点,并启动查看。

```sh
cd ../b
./generator --build_install_package ./meta/peersa.txt ./nodeb
bash ./nodeb/start_all.sh
ps -ef | grep fisco
```

看到以下节点信息，代表所有节点已经启动成功。

![image.png](https://s2.loli.net/2025/03/16/DF5R6fXMKVdhygA.png)

10.以上区块链搭建完成，但还需要可视化界面。

```sh
git clone https://gitee.com/BackRe/webase.git #现在群里下
```

部署

```sh
python3 deploy.py installAll
```

输出信息中如若包含re-download的都是选择n，其他的都是y

启动

```sh
python3 deploy.py startAll
```

报告项目经理按你下达的任务指令我以完成部署工作。

所以我将我的区块链ip等信息交给项目经理。
