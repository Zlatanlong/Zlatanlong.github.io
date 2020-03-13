---
title: (转)vue+axios上传文件
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2018-09-04 17:07:03
updated: 2018-09-04 17:07:03
tags:
  - iView
  - axios
  - 上传文件
categories:
  - Vue.js
---

## 单独上传文件：

```html
<input class="file" name="file" type="file" accept="image/png,image/gif,image/jpeg" @change="update"/>
```

<!-- more -->

```javascript
methods: {
      update(e){
        let file = e.target.files[0];
        let param = new FormData(); //创建form对象
        param.append('file',file);//通过append向form对象添加数据
        console.log(param.get('file')); //FormData私有类对象，访问不到，可以通过get判断值是否传进去
        let config = {
          headers:{'Content-Type':'multipart/form-data'}
        }; //添加请求头
        this.$http.post('http://127.0.0.1:8081/upload',param,config)
          .then(response=>{
            console.log(response.data);
          })
      }
    }
```



## Form表单上传文件：

```html
<form>
    <input type="text" value="" v-model="name" placeholder="请输入用户名">
    <input type="text" value="" v-model="age" placeholder="请输入年龄">
    <input type="file" @change="getFile($event)">
    <button @click="submitForm($event)">提交</button>
</form>
```



```javascript
data: {
      name: '',
      age: '',
      file: ''
    },
    methods: {
      getFile(event) {
        this.file = event.target.files[0];
        console.log(this.file);
      },
      submitForm(event) {
        event.preventDefault();
        let formData = new FormData();
        formData.append('name', this.name);
        formData.append('age', this.age);
        formData.append('file', this.file);
 
        let config = {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        }
 
        this.$http.post('http://127.0.0.1:8081/upload', formData, config).then(function (response) {
          if (response.status === 200) {
            console.log(response.data);
          }
        })
      }
    }
```

>版权声明：本文为CSDN博主「帅气的梧桐述」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
>原文链接：https://blog.csdn.net/h363659487/article/details/79035388