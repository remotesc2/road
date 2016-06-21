---
title: Express4xx不懂（二）-- mongodb
date: 2016-06-19 22:05:17
tags:
- express4
- mongodb
categories: 
- 学习
- back-end
- node.js
- Express4
---


从0开始用Express
===

盲点
---

### Express到底要怎么用mongodb？  

#### mongodb安装

智商不够，所以最终选择使用 brew 来安装 mongodb，指令如下  

1. `brew update`  
2. `brew install mongodb`

等待MongoDb下载完成后，会打印如下信息  

<!--more-->

```
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/mongodb-2.6
######################################################################## 100.0%
==> Pouring mongodb-2.6.5.mavericks.bottle.2.tar.gz
==> Caveats
To have launchd start mongodb at login:
ln -sfv /usr/local/opt/mongodb/*.plist ~/Library/LaunchAgents
Then to load mongodb now:
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist
Or, if you don’t want/need launchctl, you can just run:
mongod —config /usr/local/etc/mongod.conf
==> Summary
🍺 /usr/local/Cellar/mongodb/2.6.5: 17 files, 331M
dus-MacBook-Pro:CountMeInServer dudaniel$
```

然后便可使用mongodb客户端进行数据库连接，比较懒，所以没有去研究命令行操作，这里使用了 **Robomongo**  

#### 安装mongodb插件

需要在 node.js 中额外下载 **mongoose** 插件，`npm install mongoose --save-dev`  

#### 如何添加数据？

在 Express 项目里开始新建 Models 文件夹，在文件夹里新建 userModel.js，插入以下代码    

``` javascript
var mongoose = require("mongoose");
//创建模型
var Schema = mongoose.Schema;

//定义了一个新的模型，但是此模式还未和users集合有关联
var userSchema = new Schema({
    name: String,
    telephone: String
});

//与users集合关联
exports.user = mongoose.model('users', userSchema);  
```

**注意：**这里有一点，关于Collections的命名，从表面上看最好是有 **s** 后缀，如果没有，mongo 里会自动增加一个 **s** 后缀。不过这个原因并没有在百度上查找到，所以姑且只算做个表现吧，有待证实。  

然后在 c 层进行调用  

``` javascript
var mongoose = require('mongoose');
var models = require('../models/userModel.js');

var UserModel = models.user;
mongoose.connect('mongodb://localhost/conference');

router.post('/add', function (req, res, next) {
    res.setHeader('Content-Type', 'application/json;charset=utf-8');
    var userModel = new UserModel({
        name: 'AAA',
        telephone: '2032'
    });
    userModel.save(function(err){
        var state = false;
        if(err){
            //console.log(err);
        }else{
            state = true;
        }
        res.send({
            success: state
        });
    });
});
```

#### 如何查询数据？

``` javascript
var mongoose = require('mongoose');
var models = require('../models/userModel.js');

var UserModel = models.user;
mongoose.connect('mongodb://localhost/conference');

router.get('/query', function (req, res, next) {
	var id = req.query.id;
	console.log('id = ' + id);
	
	if(id && '' != id) {
	    UserModel.findById(id, function(err, docs){
			console.log('-------findById()------' + docs);
			
			res.render('modify.html',{title:'修改ToDos',demo:docs});
		});
	}
});
```

#### 如何修改数据？

``` javascript
var mongoose = require('mongoose');
var models = require('../models/userModel.js');

var UserModel = models.user;
mongoose.connect('mongodb://localhost/conference');

router.post('/modify', function (req, res, next) {
    var user = {
		name : req.body.name,
		telephone: req.body. telephone
	};
	
	// 因为是post提交，所以不用query获取id
	var id = req.body.id; 
	if(id && '' != id) {
		console.log('----update id = ' + id);
		// 这里等价于用新的 user 替换原来 _id 等于 id 的那条数据，
		// 如果 user 中缺少某个属性，数据库里对应字段会变成 null 
		UserModel.findByIdAndUpdate(id, user, function(err, docs) {
			// docs 即对应的 model 对象
			console.log('update-----'+ docs);
			res.redirect('/');
		});
	}
});
```

#### 如何删除数据？

``` javascript
var mongoose = require('mongoose');
var models = require('../models/userModel.js');

var UserModel = models.user;
mongoose.connect('mongodb://localhost/conference');

router.get('/delById', function (req, res, next) {
    var id = req.query.id;
	console.log('id = ' + id);
	
	if(id && '' != id) {
		console.log('----delete id = ' + id);
		UserModel.findByIdAndRemove(id, function(err, docs) {
			console.log('delete-----'+ docs);
			res.redirect('/');
		});
	}
});
```

参考：
---

1. [http://www.cnblogs.com/junqilian/p/4109580.html](http://www.cnblogs.com/junqilian/p/4109580.html)  
2. [http://www.bubuko.com/infodetail-1339144.html](http://www.bubuko.com/infodetail-1339144.html)  
3. [http://houzhiqingjava.blog.163.com/blog/static/167399507201311154854496/](http://houzhiqingjava.blog.163.com/blog/static/167399507201311154854496/)  
4. [http://www.nodeclass.com/api/mongoose.html#model_Model.find](http://www.nodeclass.com/api/mongoose.html#model_Model.find)