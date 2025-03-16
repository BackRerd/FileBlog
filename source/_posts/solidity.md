---
title: 智能合约工程师
date: 2025-03-16 14:28:16
tags: ['智能合约']
---

项目经理下达任务书。



收到，根据项目经理经理下达的任务书我将进行区块链核心部分操作。

1.根据运维工程师部署的可视化界面，来编写智能合约。

```sh
http://127.0.0.1:5000/  #webase界面，ip根据实际情况修改
```

2.智能合约代码编写

```solidity
pragma solidity ^0.4.25;

// 用户存储合约
contract UserStorage {
    // 用户结构体，存储哈希密码和用户是否存在的标识
    struct User {
        bytes32 passwordHash; // 存储密码的哈希值，避免明文存储，提高安全性
        bool exists; // 标识用户是否已注册
    }
    
    // 使用映射存储用户名对应的用户信息
    mapping(string => User) private users;
    
    /**
     * @dev 存储用户密码（哈希化）
     * @param _username 用户名
     * @param _password 明文密码（将被哈希存储）
     */
    function storePassword(string _username, string _password) public {
        require(!users[_username].exists, "User already registered."); // 确保用户未注册
        users[_username] = User(keccak256(abi.encodePacked(_password)), true); // 存储哈希密码
    }
    
    /**
     * @dev 验证用户密码是否正确
     * @param _username 用户名
     * @param _password 用户输入的明文密码
     * @return 是否验证成功（true: 密码匹配, false: 密码错误）
     */
    function verifyPassword(string _username, string _password) public view returns (bool) {
        require(users[_username].exists, "User not registered."); // 确保用户已注册
        return users[_username].passwordHash == keccak256(abi.encodePacked(_password)); // 校验密码哈希值
    }
}
```

3.安装区块链控制台

```sh
cd
cd ./generator
tar -zxvf ./resource/console.tar.gz -C ./
```

将sdk密钥证书复制到控制台的`conf`中,并且初始化config.toml文件

```sh
cp ./sdk_ca_a/sdk/* ./console/conf/
cp -r ./console/conf/config-example.toml ./console/conf/config.toml
```

启动控制台

```sh
bash ./console/start.sh
```

查询当前块的高度,退出控制台

```
getBlockNumber
exit
```

进入控制台文件夹下,并生成与前后端连接的代码

```sh
cd ./console
bash sol2java.sh -p site.backrer.fisco
```

出现`success`代表代码生成成功。

报告项目经理按你下达的任务指令我以完成部署工作。

我将把连接信息交给你项目经理。
