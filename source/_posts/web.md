---
title: 应用开发工程师
date: 2025-03-23 11:32:34
tags: ['web']
---

根据任务书，我需要使用SpringBoot+vue完成捐赠系统核心功能开发，实现区块链集成（区块链链接）、用户认证及管理模块扩展  (区块链测试)

首先完成任务一，基础功能开发制作出让用户登录注册的页面。

```sh
# 路径 -> (vue/src/router/index.js)
{ path:'/login',component:() => import('@/views/Login.vue')},
```

登录注册页面已经正常显示。

由于项目需要搭配智能合约去使用，所以我需要获取到智能合约的地址才能去使用智能合约。

首先我先测试一下是否能与区块链做连接。

FiscoTextOrigin.java

```java

		String path = FiscoTextOrigin.class.getClassLoader().getResource("config-fisco.toml").getPath();
		//通过Fisco提供的类，对区块链网络连接进行构建
        BcosSDK sdk = BcosSDK.build(path);
		//构建完成后，需要对区块链网络的连接，并且获取连接信息
        Client client = sdk.getClient(1);
		//获取到连接信息后，获取一下块的高度测试一下是否正常连接
        System.out.println("当前区块高度 = " + client.getBlockNumber().getBlockNumber());
		//通过智能合约工程师编写的SDK，我对合约进行一下部署
        UserStorage userStorage = UserStorage.deploy(client,client.getCryptoSuite().getCryptoKeyPair());
		//部署后获取其部署的合约地址
        System.out.println("智能合约部署地址 = " + userStorage.getContractAddress());
		//并测试一下合约功能是否正常
        userStorage.storePassword("zhangsan","123456");
        System.out.println("登陆返回信息 = " + userStorage.verifyPassword("zhangsan", "123456"));
```

进行运行启动，这里以完成区块链的连接，连接已经正常，并获取到了智能合约地址，我将对智能合约地址进行设置。



接下来我将完成智能合约与登录功能的连接，使用智能合约中的自动化用户管理功能。

WebController.java

```java
/**
     * 注册
     */
    @PostMapping("/register")
    public Result register(@RequestBody Student student) {
        studentService.add(student);
        FiscoConnect.userStorage.storePassword(student.getUsername(),student.getPassword()); //只写这一条就可以了
        return Result.success();
    }
```

我负责应用核心功能开发，完成了文字AI对接测试及区块链集成任务，同时发现了AI连接不稳定、区块链接口不兼容及应用性能不足等问题。并且我已找到相应的应对措施，现我将相关信息交给项目经理

(项目经理测试)



2.0版本

制作用户反馈功能

Manager.vue

```vue
<el-menu-item index="/manager/studentFeedback" v-if="data.user.role === 'STUDENT'">
            <el-icon><ChatLineRound /></el-icon>
            <span>用户反馈</span>
          </el-menu-item>
```

新增捐赠类型

```sql
INSERT INTO `donate_type` VALUES (8, '其他');
```

新增数据回溯(manger/student.vue)

```vue
<el-button type="danger" circle @click="reload(scope.row.id)">回</el-button>
```

新增AI帮助回复功能,通过最新的开源大模型DeepseakR1-671B自行训练的蒸馏模型API进行对接(/deepseek/chat/{msg})。

Manger.vue

```js
messages.value.push({ text: res.data.content,sender: 'bot'})
handleNavigation(res.data.match)
```

