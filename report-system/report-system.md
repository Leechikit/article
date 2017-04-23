# 接口报警系统

## 前言
在工作中经常会遇到这样的场景，用户保障页面空白或者某一操作失败，一般只会提示“请重新刷新”，“稍后重试”，“服务器故障”，而不会告诉用户具体接口的错误原因，导致保障时前端无法准确判别是哪个接口返回了什么错误码，而且从保障到反应到技术人员往往需要较长的时间。
因此需要一个可以及时收到用户因接口错误保障的信息（通过邮件通知是个不错的选择），并且能够准确知道是哪个接口出问题了，具体错误是什么。

## 前期准备
* 服务端使用`Node.js`，框架使用`Koa`
* 数据库使用`mongodb`
* 邮件模块`nodemailer`和`nodemailer-smtp-transport`

## 构建项目
首先使用[koa-generator](https://github.com/17koa/koa-generator)来构建项目。
### 1 安装koa-generator
在终端输入：
```
$ npm install -g koa-generator
```

### 2 使用koa-generator生成koa2项目
在工作目录下，输入：
```
$ koa2 reportSystem
```

成功创建项目后，进入项目目录，并执行`npm install`命令
```
$ cd reportSystem 
$ npm install
```

### 3 启动项目
在终端输入：
```
$ npm start
```
项目启动后，默认端口号是3000，在浏览器能访问`http://127.0.0.1:3000/`证明启动成功。

## 日志入库
把接受到的错误信息存储到数据库，日后可制作成图表，方便统计分析接口报警率，频发时段等信息。

### 1 安装mongodb和mongoose
在项目目录，输入
```
npm i moogodb mongoose --save
```

## 路由
要接受到错误信息，首先要提供在后端接口错误时请求的接口，把接口url、接口返回的错误码等信息作为接口的参数传送到我们的报警系统。

我们希望接口的地址是这要的
```
127.0.0.1:3000/log/w
```
在*routes*目录下添加**log.js**文件，代码如下：
```
const router = require('koa-router')();
const report = require('../utils/report');

router.get('/w', async (ctx, next)=>{
    ctx.body = 'success';
    await report();
});

module.exports = router;
```
这样就完成了*w*进行了路由配置。

接下来我们要对*log*进行路由配置，在*api.js*文件中添加以下代码：
```
const log = require('./routes/log');
......
router.use('/log', log.routes(), log.allowedMethods());
```

启动服务，在浏览器中访问127.0.0.1:3001/api/users/getUser可以得到如下输出，说明配置成功。

## 发送邮件
把报警的信息制作成表格，发送至配置的邮箱。

### 1 发送邮件配置文件
在项目目录下创建*config*目录，放置配置文件，在*config*目录下添加*host_config.js*，代码如下：
```
module.exports = {
    user: 'xxx@gmail.com', // 发送邮件邮箱
    pass: 'xxx' // 密码
}
```

### 2 封装发送邮件模块
在*utils*目录下添加*send_mail.js*，代码如下：
```
const nodemailer = require('nodemailer');
const smtpTransport = require('nodemailer-smtp-transport');
const config = require('../config/host_config');

let transport = nodemailer.createTransport(smtpTransport({
    host: "smtp.gmail.com",
    port: 465,
    auth: {
        user: config.user,
        pass: config.pass
    }
}));

let sendMail = (receiver, theme, html) => {
    return new Promise((resolve,reject)=>{
        transport.sendMail({
            from: config.user,
            to: receiver,
            subject: theme,
            html: html
        }, (err, res) => {
            if (err) {
                console.log(err);
            } else {
                console.log('发送成功');
            }
        });
    });
}

module.exports = sendMail;
```

## 报警条件
通过配置，可订阅接口，定时发邮件通知配置时间段内接口报警情况。
还可以在一段时间内接口报警次数超过配置阀值时发邮件通知。

### 1 接收邮件配置文件
在*config*目录下添加*report_config.js*，代码如下：
```
module.exports = {
    1: { // 项目id
        name: '', // 项目名称
        watch: true, // 是否监听
        checktime: 3600000, // 检查报警数量间隔时间毫秒
        reportLimit: 3, // 报警阀值
        reportDay: 1, // 自动邮件通知间隔天数
        emails: [ // 报警通知邮箱
            'xxx@qq.com',
            'yyy@qq.com'
        ]
    }
}
```
### 2 创建日志模型
创建日志模型前，需要链接数据库，并创建链接，定义logSchema（相当于数据库建表），代码如下：
```
const mongoose = require('mongoose');

let Schema = mongoose.Schema;
let LogSchema = new Schema({
    reportType: String,
    data: Object
});

mongoose.connect('mongodb://127.0.0.1/logs');

mongoose.connection.on('connected', function () {
    console.log('Mongoose connection open');
});

export default mongoose.model('errorreport', LogSchema);
```

> 需要注意的是使用*mongoose*创建模型时，若代码中的集合名是单数时，在数据库中对应的集合名是复数；若代码中的集合名是复数时，在数据库中对应的集合名则相同。

