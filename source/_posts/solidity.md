---
title: 智能合约工程师
date: 2025-03-16 14:28:16
tags: ['智能合约']
---

项目经理下达任务书。

收到，根据项目经理经理下达的任务书我将使用Soliolidity完成区块链智能合约开发、JavaSDK部署及前后端联调，实现用户密码哈希化安全存储与验证功能  

根据运维工程师部署的可视化界面，我将完成智能合约编写，为整个项目的账本分发，用户管理等做自动化处理。

```sh
http://127.0.0.1:5000/  #webase界面，ip根据实际情况修改
```

首先进行任务一，智能合约代码编写

```solidity
pragma solidity ^0.4.25;

// 用户存储合约
contract UserStorage {
    // 用户结构体，表示用户的信息
    struct User {
        bytes32 passwordHash; 
        bool exists; 
    }
    
    // 使用映射存储用户名对应的用户信息
    mapping(string => User) private users;
    
    //创建出自动化注册方法，注册时候自动对密码进行哈希加密，并自动分发到区块链中
    function storePassword(string _username, string _password) public {
        require(!users[_username].exists, "User already registered."); // 确保用户未注册
        users[_username] = User(keccak256(abi.encodePacked(_password)), true); // 存储哈希密码
    }
    
    //创建对用户密码的验证，在密码输入时，对密码自动哈希处理，对比密码是否正确。
    function verifyPassword(string _username, string _password) public view returns (bool) {
        require(users[_username].exists, "User not registered."); // 确保用户已注册
        return users[_username].passwordHash == keccak256(abi.encodePacked(_password)); // 校验密码哈希值
    }
}
```

合约编写完成，保存并编译，合约编译成功。部署合约，合约部署成功。

接下来进行任务二，区块链环境配置

由于在后续需要区块链中的委员用户等管理，所以我将初始化一下控制台，并且连接一下区块链网络。

```sh
cd
cd ./generator
tar -zxvf ./resource/console.tar.gz -C ./
```

```sh
cp ./sdk_ca_a/sdk/* ./console/conf/
cp -r ./console/conf/config-example.toml ./console/conf/config.toml
```

```sh
bash ./console/start.sh
```

控制台正常启动后，我需要测试一下是否能正常使用，所以我将进行任务三，委员账号的添加与管理。

(正常操作委员部分)

.........

最后完成任务四，系统联调

智能合约自动化部署需要与其他语言进行连接，所以我需要生成对应语言的SDK代码创建。

```sh
cd ./console
bash sol2java.sh -p site.backrer.fisco
```

我负责智能合约的开发与部署，完成了合约源码、部署日志、Java适配代码及测试报告的交付，同时发现了密码算法单一、标识管理混乱及路径配置不一致等问题。
