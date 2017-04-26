# 接口报警系统

## 前言
在工作中经常会遇到这样的场景，用户保障页面空白或者某一操作失败，一般只会提示“请重新刷新”，“稍后重试”，“服务器故障”，而不会告诉用户具体接口的错误原因，导致保障时前端无法准确判别是哪个接口返回了什么错误码，而且从保障到反应到技术人员往往需要较长的时间。
因此需要一个可以及时收到用户因接口错误保障的信息（通过邮件通知是个不错的选择），并且能够准确知道是哪个接口出问题了，具体错误是什么。

## 前期准备
* 服务端使用`Node.js`，框架使用`Koa`
* 数据库使用`mongodb`
* 邮件模块`nodemailer`和`nodemailer-smtp-transport`

## 构建项目

[项目仓库](https://github.com/Leechikit/report-system)

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
项目启动后，默认端口号是3000，在浏览器能访问 *http://127.0.0.1:3000/* 证明启动成功。

## 路由
要接受到错误信息，首先要提供在后端接口错误时请求的接口，把接口url、接口返回的错误码等信息作为接口的参数传送到我们的报警系统。

## 1 设置路由
我们希望接口的地址是这要的
```
127.0.0.1:3000/log/w
```
在*routes*目录下添加**log.js**文件，代码如下：
```
const router = require('koa-router')();

router.get('/w', async (ctx, next)=>{
    ctx.body = 'success';
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

启动服务，在浏览器中访问 *http://127.0.0.1:3001/log/w* 可以得到如下输出，说明配置成功。

## 2 封装上报接口
封装一个上报接口，方便用户在接口返回结果中调用，并传入各种参数上报。

在*public > javascripts*目录中添加**report.js**文件，添加代码如下：
```
const url = "/log/w";

let getParam = (opt)=>{
    let obj = opt;
    var param = [];
    for( let h in obj){
        if(obj.hasOwnProperty(h)){
            param.push(encodeURIComponent(h)+"="+(obj[h] === void 0 || obj[h] === null ? "": encodeURIComponent(obj[h])));
        }
    }
    return param.join("&");
}

let reportEvent = (reportType,reportValue,appVersion,phoneType) => {
    let param = getParam({
        reportType,
        reportValue,
        appVersion,
        phoneType
    })
    let img = new Image();
    img.src =  url + "?" + param;
}
```

在需要上报接口错误的页面引入 *http://127.0.0.1:3000/javascripts/report.js* 文件，使用`reportEvent`方法上报。

## 日志入库
把接受到的错误信息存储到数据库，日后可制作成图表，方便统计分析接口报警率，频发时段等信息。

### 1 创建数据库*logs*和集合*errorreports*

### 2 安装mongodb和mongoose
在项目目录，输入
```
npm i moogodb mongoose --save
```

### 3 创建日志模型
创建日志模型前，需要链接数据库，并创建链接，定义logSchema（相当于数据库建表）。在*utils*目录下添加**log_model.js**，代码如下：
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

module.exports = mongoose.model('errorreport', LogSchema);
```

> 需要注意的是使用*mongoose*创建模型时，若代码中的集合名是单数时，在数据库中对应的集合名是复数；若代码中的集合名是复数时，在数据库中对应的集合名则相同。

### 4 格式化上报的日志
对路由获取的数据进行格式化。在*utils*目录下添加**log_format.js**，代码如下：
```
/**
 * 格式化上报日志
 *
 * @param: {Objet} ctx 请求对象
 * @return: {Object} 格式化后的对象
 */
var logFormat = function (ctx) {
    var logObj = new Object();
    // 请求信息
    var req = ctx.request;
    // 添加请求日志
    var method = req.method;
    // 页面地址
    logObj["url"] = req.header["referer"];
    // 上报时间
    let date = new Date();
    date.setSeconds(0);
    date.setMilliseconds(0);
    logObj["time"] = date.getTime();
    // 请求参数
    if (method === 'GET') {
        let query = req.query;
        for(let key in query){
            logObj[key] = query[key];
        };
    } else {
        logObj["request body"] = req.body;
    }
    return logObj;
}

module.exports = logFormat;
```

### 5 保存日志到数据库
在*utils*目录下添加**save_log.js**，代码如下：
```
const LogModel = require('./log_model');

/**
 * 保存日志
 *
 * @param: {Object} obj 信息
 */
function saveLog(obj){
	let log = new LogModel(obj);
	log.save((error, doc) => {
	    if (error) {
	        console.log(error);
	    }
	});
}

module.exports = saveLog;
```

### 6 上报日志
在*w*的路由设置中，获取得到的数据并格式化，然后保存到数据库中。修改*routes*目录下的**log.js**文件，代码如下：
```
const router = require('koa-router')();
const logFormat = require('../utils/log_format');
const saveLog = require('../utils/save_log');

router.get('/w', async (ctx, next)=>{
    ctx.body = 'success';
    // 格式化获取的数据
    let logObj = logFormat(ctx);
    // 保存到数据库
    await saveLog(logObj);
});

module.exports = router;
```

## 发送邮件
把报警的信息制作成表格，发送至配置的邮箱。

### 1 发送邮件配置文件
在项目目录下创建*config*目录，放置配置文件。在*config*目录下添加**host_config.js**，代码如下：
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

/**
 * 发送邮件
 *
 * @param: {String} receiver 接收邮箱
 * @param: {String} theme 邮箱标题
 * @param: {String} html 邮箱内容
 */
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

## 定时上报信息
通过配置，可订阅接口，定时发邮件通知配置时间段内接口报警情况。

### 1 接收邮件配置文件
在*config*目录下添加**report_config.js**，代码如下：
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

### 2 查找数据库
在*utils*目录下添加**find_log.js**，代码如下：
```
/**
 * 查询数据库
 *
 * @param: {Object} query 查询条件
 * @param: {Object} sort 排序条件
 * @param: {Number} limit 显示数据数
 */
function findLog(query, sort, limit) {
    return new Promise((resolve,reject)=>{ 
        LogModel.find(query, {'_id':0}, (err,res)=>{
            if(err){
                console.log(err);
            }else{
                resolve(res);
            }
        }).sort(sort).limit(limit);
    });
}
```

### 3 上报信息发送邮件
根据条件查询数据库，把获取的信息发送给配置的邮件。在*utils*目录下添加**regular_report.js**，代码如下：
```
const sendMail = require('./send_mail');
const reportConfig = require('../config/report_config');
const hostConfig = require('../config/host_config');
const mailFormat = require('./mail_format');
const findLog = require('../utils/find_log');
/**
 * 上报信息发送邮件
 *
 * @param: {Number} reportType 上报项目id
 * @param: {Date} minDate 最小日期
 * @param: {Date} maxDate 最大日期
 */
 async function reportLogMes(reportType,minDate,maxDate) {
 	// 获取数据
    let mes = await findLog({ 'reportType': reportType, 'time': { '$gt': minDate.time, '$lte': maxDate.time } });
    // 邮件标题
    let theme = getTheme(reportType,minDate,maxDate);
    // 配置的邮件
    let {mails} = reportConfig[reportType];
    // 数据库中有数据则发送
    mes && mails.forEach((item) => {
        sendMail(item,theme,mailFormat(mes));
    });
}
```

### 4 循环所有项目
循环配置文件中的所有项目配置，若配置了可监听并且有收件邮箱则发送邮件。在**regular_report.js**中添加代码如下：
```
/**
 * 循环所有项目
 *
 * @param: {Date} minDate 最小日期
 * @param: {Date} maxDate 最大日期
 */
function loopReportConfig(minDate, maxDate) {
    for (let reportType in reportConfig) {
        let {
            watch,
            mails
        } = reportConfig[reportType];
        // 若开启监听并有设置邮箱则发送邮件
        if (watch == true && mails && mails.length > 0) {
            reportLogMes(reportType,minDate,maxDate);
        }
    }
}
```

### 5 定时轮询
定时轮询，当配置时间与当前时间相同时则开始循环所有项目配置，和上报信息发送邮件。在**regular_report.js**中添加代码如下：
```
// 轮询间隔
const DURATION = 60000;
/**
 * 定时轮询
 *
 */
function loopDate() {
	//获取设置小时和分钟
    let {configHour, configMinute} = getConfigDate();
    let interval = setInterval(() => {
    	// 获取当前小时,分钟和日期
        let nowDate = getNowDate();
        // 当设置时分与当前时分一样，则循环项目发送邮件
        if (configHour == nowDate.hour && configMinute == nowDate.minute) {
            // 获取上一天日期
            let lastDate = getLastDate()
            // 循环所有项目
            loopReportConfig(lastDate, nowDate);
        }
    }, DURATION);
}

```

## 实时报警
在配置时间内接口日志次数达到配置阀值时发邮件通知。

怎么计算配置时间内接口日志次数是一个难点。

1. 方法一是给各个项目创建一个计算日志数的计数器，然后每个项目根据配置时间轮询，每次轮询检查计数器是否达到配置阀值并清空计数器，达到则发邮件通知。这种方法一是需要多个轮询，在接口稳定长时间没有上报日志时显然浪费资源；二是轮询间隔时间固定，若在第一个时间段后半段报警数是阀值的一半，在第二个时间段前半段报警数是阀值的一半，虽然在两次时间段内都不会报警，但是第一个时间段后半段和第二个时间段前半段相加是在配置时间内的，报警数相加达到了阀值，应该是一个需要报警的情况。
2. 方法二是每次有日志上报时，到数据库内查询配置时间内的日志数，若日志数达到配置阀值则发邮件通知。这种方法一是在频繁上报日志的情况下会多次查询数据库造成数据库压力过大；二是会在同一时间段内重复报警。
3. 方法三是把各个项目日志日期倒序并把配置阀值数量的条数缓存起来，每次有日志上报时，把日志的日期加到缓存队列的最后，并保持队列数量不超过设置的阀值，超过则把队列最前面的删除。然后检查队列中配置时间内的日志数量是否达到配置阀值，达到则发邮件通知。

上面3种方法中，第三种方法对数据库压力最少，也不需要一直轮询，因此以下我们按照方法三的思路来展开代码。

### 1 缓存项目日志的日期
把项目的日志时间缓存到对象`dateCache`里。在*utils*目录下添加**realtime_report.js**，代码如下：
```
// 日期缓存
let dateCache = {};

/**
 * 保存日志的日期到缓存中
 *
 * @param: {Object} logs 日志对象
 */
function saveDate(logs){
    logs.forEach((log)=>{
        dateCache[log._doc.reportType].unshift(log._doc.time);
    });
}
```

### 2 缓存条件
按日志时间倒叙排列对应项目日志，取出设置阀值数量的日志，若日志存在则缓存，在**realtime_report.js**中添加代码如下：
```
const findLog = require('./find_log');

/**
 * 初始化日期缓存
 *
 * @param: {String} reportType 项目id
 * @param: {Number} reportLimit 报警阀值
 */
async function initDate(reportType, reportLimit){
    let logs = await findLog({'reportType':reportType},{'time':-1},reportLimit);
    // 日志数量大于0则存储到缓存中
    await logs.length > 0 && saveDate(logs);
}
```

### 3 循环所有项目
循环配置文件中的所有项目配置，若配置了可监听则缓存日期，在**realtime_report.js**中添加代码如下：
```
const reportConfig = require('../config/report_config');

function loopReportConfig() {
    for( let reportType in reportConfig){
        let {
            watch,
            reportLimit
        } = reportConfig[reportType];
        // 若开启监听，则缓存日期
        if(watch == true){
            dateCache[reportType] = [];
            initDate(reportType,reportLimit);
        }
    }
}
```

### 4 添加日期到缓存
上报日志时，需要把新日志的日期添加到缓存中，保持项目对应的日期缓存数量不超过对应的阀值，在**realtime_report.js**中添加代码如下：
```
/**
 * 添加日期到缓存
 *
 * @param: {String} reportType 项目id
 * @param: {Number} time 日期
 */
function addDate(reportType, time){
	// 添加日期到缓存中
    dateCache[reportType].push(time);
    // 若缓存中的日期数量大于配置的阀值，则删除最早的日期。
    if(dateCache[reportType].length > reportConfig[reportType].reportLimit){
        dateCache[reportType].shift();
    }
}
```

### 5 获取检查时间内日志数量
上报日志时，需要检查从缓存最后一个日期往前，到设置的检查时间内的缓存日期数量，在**realtime_report.js**中添加代码如下：
```
/**
 * 获取设置的检查时间内日志数量
 *
 * @param: {String} reportType 项目id
 * @return: {Number} 日志数量
 */
function countLog(reportType){
    let checktime = reportConfig[reportType]['checktime'];
    let dateList = dateCache[reportType];
    // 缓存中项目的最后一个日期
    let lastDate = dateList[dateList.length - 1];
    // 获取从最后一个日期往前，设置的检查时间内的缓存日期。
    let filterDate = dateList.filter((date)=>{
        return +date >= +lastDate - +checktime;
    });
    return filterDate.length;
}
```

### 6 实时上报
上报日志时，首先把日期添加到缓存中，然后检查从缓存最后一个日期往前，到设置的检查时间内的缓存日期数量，若数量大于或等于设置的阀值，则需要通过发送邮件报警，在**realtime_report.js**中添加代码如下：

```
const sendMail = require('./send_mail');

/**
 * 实时上报
 *
 * @param: {String} reportType 项目id
 * @param: {Number} time 日期
 */
function realtimeReport(reportType, time){
	// 添加日期到缓存
    addDate(reportType,time);
    // 获取设置的检查时间内日志数量
    let count = countLog(reportType);
    let theme = reportConfig[reportType]['name'] + '报警通知';
    let html = `<p>${reportConfig[reportType]['name']}报障数超过${reportConfig[reportType]['reportLimit']}条</p>`;
    // 若检查时间内日志数量大于设置的阀值，则发邮局报警报警
    if(count >= reportConfig[reportType]['reportLimit']){
        let mails = reportConfig[reportType]['mails'];
        mails.forEach((item) => {
            sendMail(item, theme, html);
        });
    }
}
```