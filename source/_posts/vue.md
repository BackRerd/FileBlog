---
title: vue
date: 2024-11-17 16:17:15
tags: ['vue']
---



# vue基础结构

```vue
<script lang="ts">
</script>

<template>
</template>

<style scoped>
</style>

```

## 双向绑定

````vue
<script lang="ts">
  export default{
    data(){
      return{
        userName: 'roy',
        salary:15000
      }
    },
    methods:{
      addSalary(){
        this.salary += 1000
      }
    }
  }
</script>

<template>
  <div>
    姓名：<input v-model="userName" />{{ userName }} <br>
    薪水： <input type="number" v-model="salary"/>{{ salary }}<br>
    <button v-on:click="addSalary">提交</button>
  </div>

</template>

<style scoped>

</style>

````

通过script内脚本进行编写`export default{}`内编写各种方法。

比如:

```vue
data(){
      return{
        userName: 'roy',
        salary:15000
      }
    }
```

这个方法定义了两个变量，分别返回名称和金钱

以及方法的定义:

```vue
methods:{
      addSalary(){
        this.salary += 1000
      }
    }
  }
```

`tmplate`中进行编写网页样式模板，vue中带了获取变量的方法`v-model` 获取变量`v-on:click`获取方法

## 一个较为完整的双向绑定案例

```vue
<script lang="ts">
  export default{
    data(){
      return{
        userName: 'roy',
        salary:15000,
        userInfo:{
          age:0,
          sex:1,
          department:'',
          skills:['java','python','vue']
        },
        newSkill: '',
        showUserInfo: false
      }
    },
    methods:{
      addSalary(){
        this.salary += 1000
      },
      learnNewSkill(){
        if(this.newSkill){
          this.userInfo.skills.push(this.newSkill)
        }
      },
      changeShowUserInfo(){
        this.showUserInfo = !this.showUserInfo 
      }
    }
  }
</script>

<template>
  <div>
    姓名：<input v-model="userName" />{{ userName }} <br>
    薪水： <input type="number" v-model="salary"/>{{ salary }}<br>
    <button v-on:click="addSalary">提交</button>
    <button @click="changeShowUserInfo">查看个人信息</button>
    <hr />
    <div class="userInfo" v-show="showUserInfo">
      <h2>个人信息</h2>
      <p>年龄: <input type="number" v-model="userInfo.age"></p>
      <p>性别：<input type="radio" value="1" v-model="userInfo.sex">男</input><input type="radio" value="2">女</input></p>
      <p>岗位：<select v-model="userInfo.department">
        <option value="dev">开发</option>
        <option value="test">测试</option>
        <option value="maintain">运维</option>
      </select></p>
      <p>技术： <span v-for="skill in userInfo.skills" v-bind:key="skill">{{ skill }}</span></p>
      <p>学习新技术<input type="text" v-model="newSkill"/><button @click="learnNewSkill">学习新技术</button></p>
      <p>个人信息汇总{{ userInfo }}</p>
    </div>
  </div>
</template>

<!-- scoped表示样式就在这个写 -->
<style scoped> 
  .userInfo{
    background-color: black;
    widows: 80%;
  }
  .userInfo span{
    background-color: rgb(73, 73, 5);
    margin-left: 10px;
    border: 1px;
    border-radius: 5px;
  }
</style>

```

