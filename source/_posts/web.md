---
title: 应用开发工程师
date: 2025-03-23 11:32:34
tags: ['web']
---

根据任务书，我需要完成注册登陆以及区块链的链接。

首先将首页面制作出。

```sh
# 路径 -> (vue/src/router/index.js)
{ path:'/login',component:() => import('@/views/Login.vue')},
```

path对应一下页面的login标签，然后使用component对应页面文件的位置

(打开一下界面)

界面功能编写完成，接下来测试区块链的链接。

使用已有的测试类

FiscoTextOrigin.java

```java
		//通过当前类加载器获取配置文件路径
		String path = FiscoTextOrigin.class.getClassLoader().getResource("config-fisco.toml").getPath();
        //使用bcocsdk类进行对区块链的连接，此处需要对应一个配置文件地址，使用类加载器获取加载位置
        BcosSDK sdk = BcosSDK.build(path);
        //获取区块链的client客户端.
        Client client = sdk.getClient(1);
        //获取区块高度
        System.out.println("当前区块高度 = " + client.getBlockNumber().getBlockNumber());
        //根据智能合约工程师所写代码类，进行智能合约的部署
        UserStorage userStorage = UserStorage.deploy(client,client.getCryptoSuite().getCryptoKeyPair());
        //通过部署的智能合约获取部署的地址
        System.out.println("智能合约部署地址 = " + userStorage.getContractAddress());
        //使用其中的方法创建和登陆测试zhangsan用户
        userStorage.storePassword("zhangsan","123456");
        System.out.println("登陆返回信息 = " + userStorage.verifyPassword("zhangsan", "123456"));
```

进行启动测试，测试成功

接下来将代码用于功能当中，在注册登陆等功能处添加上对应的代码

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

接下来进行页面测试，运行项目

(注册用户测试->登陆用户测试)

麻烦智能合约工程师查看一下交易或块是否增长

(增长)->报告项目经理应用开发已完成



2.0版本

制作用户反馈功能

Manager.vue

```vue
<el-menu-item index="/manager/studentFeedback" v-if="data.user.role === 'STUDENT'">
            <el-icon><ChatLineRound /></el-icon>
            <span>用户反馈</span>
          </el-menu-item>
```

新增捐赠类型，使用管理员账号进行添加新的类型

新增AI帮助回复功能,通过最新的开源大模型DeepseakR1-671B自行训练的蒸馏模型API进行对接(/deepseek/chat/{msg})。

Manger.vue

```js
const res = await request.get(`/chat/send/${encodeURIComponent(username)}/${encodeURIComponent(newMessage.value)}`)
```

