### 
1. node 与 mongoose的安装

brew install node - version
npm 自带安装完成

mongodb 安装
brew install mongodb 

第一次安装会失败。需要手动加入文件夹sudo mkdir -p /data/db 
                  需要给文件夹添加权限 sudo chown -R "user"/data 
user 是你本机的用户名


2.启动mongo
启动mongodb 
开启 mongod 
随后可以使用终端命令 mongo
3. mongodb 使用 
```
1.mongodb 不区分类型 不区分大小写 但是不能有重复键
  概念 ： db 中集合代表一个文档 对应数据库中的表
2.命令
  .1 use mongodb use data_base 
  .2 db.dropDatabase
  ... 可以查看文档
```
```
node express 搭建服务器

var express = require('express')
var app = express()
app.get('/',function(req,res){
res.send(); req.body->解析
})
app.listen(3000,function(){
})

注意 本地开启http+ localhost 才有效 不要使用www+localhost 这会有502错误

```

node express + mongoose 
```
这里就该涉及到数据库了
这个学会了那么基本上基础就ok了

使用mongoose
在一个新的model文件夹添加test.js
var mongoose = require('mongoose')
var Schema = mongoose.Schema
var userSchema = new Schema({
 name:String,
 age :Number, // 定义数据库的字段
})
说明一下 schema.type的类型 Boolean Buffer String Number Date ... 自己查去

exports.user = mongoose.model('users',userSchema)// 这里做的是创建model并且创建users这个集合。

数据库创建好了 
现在在我们app.js 也就是node的入口文件使用它
var user = require(./models/test)
引号是要的 我就不打 

app.get('/sd',function(req,res,next){
	//获取get参数
var name = req.query.name
var age = req.query.age
var datas = ['name':name,'age':age];
use.create(datas,function(err,docs){
})
这里就完成了一次数据的存储 增删改查我就不细说了可以看文档 

})


```
