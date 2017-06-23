---
layout: post
title:  use-manual
date:   2017-06-20 00:00:00 +0800
categories: document
tag: 教程
---

* content
{:toc}


 第一章 概述
====================================

conalog是集数据采集，解析，状态管理，日志管理为一体的工具，可以有效的对自编制脚本进行管理和监控，
也可以监控orientsoft软件组件的状态,主要模块分为五部分：cert，collector，parser，status，history。


 第二章 部署与安装
====================================

 2.1 环境要求
---------------------

Linux版本：^3.0  
Node版本：6.9 (LTS)  
Mongo版本：^3.0  
Redis版本：^3.0

 2.2 前台
---------------------

1. 获取conglog-front代码；
2. 修改/conalog-front/config 目录下的config.js 文件，将conalogHost的值改为虚拟机的IP地址；

``` 
conalogHost:’192.168.0.244’;
```
``` 
//config.js配置文件
var config = {
  logLevel: 'info',
  conalogHost: '192.168.0.244',
  conalogPort: 17527,
  conalogFrontPort: 7527,
  mongoUrl: 'mongodb://127.0.0.1:27017/conalog',
  redisUrl: 'redis://127.0.0.1:6379',
  activeCollectorPrefix: 'ac_',
  passiveCollectorPrefix: 'pc_',
  apiGatewayHost: 'apigateway',
  apiGatewayPort: 1234,
  apiGatewayToken: '12345',
  apiGatewayUid: '67890',
  apiGatewayType: 'user'
}
module.exports = config;
```

3. 安装模块：npm i；
4. 执行：gulp install；
5. 编译：gulp go；
6. 启动：npm start；

2.3 后台
---------------------

1. 获取conalog代码;
2. 修改/ conalog /config 目录下的config.js 文件，将conalogHost，conalogFrontHost 以及nanomsgHost 的值改为虚拟机的IP地址；

``` 
conalogHost:’192.168.0.244’;
conalogFrontHost:’192.168.0.244’;
nanomsgHost:’192.168.0.244’;
```
``` 
//config.js配置文件
var config = {
  logLevel: 'info',
  logLifespan: 604800000, // 7 days, in ms
  userAuthSwitch: false,
  conalogHost: '192.168.0.244',
  conalogPort: 17527,
  conalogFrontHost: '192.168.0.244',
  conalogFrontPort: 7527,
  mongoUrl: 'mongodb://127.0.0.1:27017/conalog',
  redisUrl: 'redis://127.0.0.1:6379',
  nanomsgProtocol: 'tcp://',
  nanomsgHost: '192.168.0.244',
  nanomsgRequestPort: 17537,
  nanomsgStartPort: 17627,
  nanomsgEndPort: 17927,
  filebeatChannel: 'conalog-filebeat',
  parserPathPrefix: './parser/'
}
module.exports = config;
```
3. 安装模块：npm i；
4. 执行：gulp install；
5. 编译：gulp go；
6. 启动：node bin/www；  

 2.4 常见问题
---------------------

-  每次修改代码后要运行 gulp go 使修改生效;
-  执行gulp install报错 "无法找到gulp" 时即执行 'npm install -g gulp'；
-  运行npm i命令后遇到缺少模块的问题时，npm install 该模块；

 2.5 访问网址
---------------------

conalogHost:conalogFrontPort  
如：'192.168.0.244:7527'  

登录界面：（账号：admin 初始密码：admininitpass）
![](/conalog-doc/styles/images/logIn.png)

登录成功：
![](/conalog-doc/styles/images/homePage.png)


 第三章 cert
====================================

cert功能：存储用于远程SSH登录的账号。Conalog可以通过ssh连接登陆虚拟机，随后可以执行Shell命令。

3.1 添加
---------------------

1. 点击左上角添加按钮：
![](/conalog-doc/styles/images/addCert.png)
2. 弹出添加框，填写信息：  
   填写规范：    
   Host: 192.168.0.244，（虚拟机IP地址）  
   Port：22，（端口号）  
   User：voyager，（虚拟机用户名）  
   Password：welcome1，（虚拟机用户名对应的密码）
![](/conalog-doc/styles/images/addCertModal.png)

3. 添加成功：
![](/conalog-doc/styles/images/addCertSuccess.png)

3.2 修改
---------------------

1. 点击edit按钮：
![](/conalog-doc/styles/images/editCert.png)
2. 弹出修改框，修改信息：
![](/conalog-doc/styles/images/editCertModal.png)
3. 保存即点击确认，不保存即点击取消；

3.3 删除
---------------------

1. 点击delete按钮： 
![](/conalog-doc/styles/images/deleteCert.png)
2. 弹出确定框，点击确认即删除，点击取消即取消删除：
![](/conalog-doc/styles/images/deleteCertModal.png)

3.4 查看密码
---------------------

1. 鼠标放在图标上，password即显示为明文密码，鼠标移开，密码即为隐藏密码；
![](/conalog-doc/styles/images/checkPassword.png)


第四章 collector
====================================

collector作用：执行命令，采集数据。  
分为active、passive和agent collector三种类型：  
1. active collector是根据设定的时间间隔/每天固定时间点/单次执行命令，一般用于编写脚本主动读取日志；  
2. passive collector是执行一次命令并一直保持执行状态，通常用于tail -F等长期执行的任务；  
3. agent collector是从Filebeat统一的日志更新通道接收数据，根据文件通配符规则，把同类型的日志，分发到指定的通道中。  

 4.1 添加
---------------------

1. active collector：  
填写规范：  
Name: 无要求（**输出数据的redis通道名默认为 ac_name**）;  
Type:  
&nbsp; &nbsp; &nbsp; &nbsp;interval : 每间隔一段时间执行一次命令；  
&nbsp; &nbsp; &nbsp; &nbsp;time ：每天定点执行命令；  
&nbsp; &nbsp; &nbsp; &nbsp;oneshot：只执行一次；  
Trigger: 对应type类型设置时间；  
Command: 执行命令；  
Parameter: 参数；  
Host: 虚拟机IP；  
Encoding: 根据电脑系统选择对应的编码；  
Channel: redis/nanomsg;  
Description: collector usage & source & description；
![](/conalog-doc/styles/images/addActiveCollector.png)
2. passive collector：  
填写规范：  
Name: 无要求（**输出数据的redis通道名默认为 pc_name**);  
Type:  
&nbsp; &nbsp; &nbsp; &nbsp;LongScript: 执行用户指定的命令；  
&nbsp; &nbsp; &nbsp; &nbsp;File Tail: 执行 tail -F 命令（快捷方式）；  
Command: 执行命令；  
Parameter: 参数；  
Host: 虚拟机IP；  
Encoding: 根据电脑系统选择对应的编码；  
Channel: redis/nanomsg;  
Description: collector usage&source&description；
![](/conalog-doc/styles/images/addPassiveCollector.png)
3. agent collector  
&nbsp;3.1 点击添加按钮：
![](/conalog-doc/styles/images/addAgentCollector.png)
&nbsp;3.2 弹出添加框，填写信息：  
&nbsp;&nbsp;&nbsp;填写规范  
&nbsp;&nbsp;&nbsp;Name: 无要求 （**输出数据的redis通道名默认为 agt_name**）；  
&nbsp;&nbsp;&nbsp;Parameter: 文件名的正则表达式；  
&nbsp;&nbsp;&nbsp;Encoding: 根据电脑系统选择对应的编码；  
&nbsp;&nbsp;&nbsp;Channel: redis/nanomsg;  
&nbsp;&nbsp;&nbsp;Description: collector usage & source & description；
![](/conalog-doc/styles/images/addAgentCollectorContent.png)

  4.2 修改
---------------------

1. active collector：  
   勾选要修改的项，再点击edit按钮，在页面上方即可出现对应的信息，修改之后保存点击save按钮，不保存点击clear按钮；
![](/conalog-doc/styles/images/editActiveCollector.png)
![](/conalog-doc/styles/images/editActiveCollectorContent.png)
2. passive collector：  
   勾选要修改的项，再点击edit按钮，在页面上方即可出现对应的信息，修改之后保存点击save按钮，不保存点击clear按钮；
![](/conalog-doc/styles/images/editPassiveCollector.png)
![](/conalog-doc/styles/images/editPassiveCollectorContent.png)
3. agent collector:  
   点击edit按钮，即会弹出修改框，修改之后保存点击确定按钮，不保存点击取消按钮；
![](/conalog-doc/styles/images/editAgentCollector.png)
![](/conalog-doc/styles/images/editAgentCollectorContent.png)

  4.3 删除
---------------------

1. active collector：  
   勾选要删除的项，再点击delete按钮，弹出确定框，删除点击确定，不删除点击取消；
![](/conalog-doc/styles/images/editActiveCollector.png)
![](/conalog-doc/styles/images/deleteActiveCollector.png)
2. passive collector：  
   勾选要删除的项，再点击delete按钮，弹出确定框，删除点击确定，不删除点击取消；
![](/conalog-doc/styles/images/editPassiveCollector.png)
![](/conalog-doc/styles/images/deletePassiveCollector.png)
3. agent collector:  
   点击delete按钮，弹出确定框，删除点击确定，不删除点击取消；
![](/conalog-doc/styles/images/deleteAgentCollector.png)
![](/conalog-doc/styles/images/deleteAgentCollectorModal.png)


第五章 parser
====================================

parser的功能：parser通过调用脚本把文件中的文本数据转换成结构数据，同一个parser可以同时启动多个实例。

  5.1 添加
---------------------

1. 点击左上角添加按钮：
![](/conalog-doc/styles/images/addParser.png)
2. 弹出添加框，填写内容，所有选项均为必填：  
填写规范：  
Name：esb   (无要求)；  
Path：esb.js   (parser脚本的路径，可以使用相对路径，默认当前目录为CONALOG_PATH/parser/)；  
Parameter：esb=1   (脚本对应的参数)；  
InputChannel：ac\_mobile (输入数据通道名)；  
OutputChannel：esb (输出数据通道名，**Conalog会自动给输出通道名加前缀p\_**)；  
InputType：RedisChannel   (RedisChannel/NanomsgQueue);  
OutputType：RedisChannel   (RedisChannel/NanomsgQueue);  
Remark：input:... output:{...}   (parser脚本作用描述，输入输出数据格式等);
![](/conalog-doc/styles/images/addParserContent.png)
3. 添加成功：
![](/conalog-doc/styles/images/addParserSuccess.png)

  5.2 修改
---------------------

1. 点击edit按钮：
![](/conalog-doc/styles/images/editParser.png)
2. 弹出修改框，修改信息：
![](/conalog-doc/styles/images/editParserModal.png)
3. 保存即点击确认，不保存即点击取消；

  5.3 删除
---------------------

1. 点击delete按钮： 
![](/conalog-doc/styles/images/deleteParser.png)
2. 弹出确定框，点击确认即删除，点击取消即取消删除：
![](/conalog-doc/styles/images/deleteParserModal.png)

  5.4 Parser脚本
---------------------

1. Parser脚本用ES2015编写，默认存放在CONALOG_PATH/parser-src目录下；  
2. 脚本编写完成后，在conalog主目录执行gulp compile-parser即可将ES2015的Parser编译成ES5代码，存放在CONALOG_PATH/parser目录下；  
3. 请参考[Conalog Parser开发指南](https://github.com/Orientsoft/conalog/wiki/ParserDev)进行Parser的开发；  
4. 可以用Git为每个部署项目开新分支，并提交新开发的parser到github conalog项目，分支准备好后请在conalog项目提交Pull Request。请咨询[项目管理员](https://github.com/xiedidan)获取写入权限。  


第六章 status
====================================

status作用：展示active collector，passive collector，agent collector以及parser执行的状态和结果，可以控制启停。

  6.1 active ／passive／agent status
---------------------

  6.1.1 控制
点击operation下的切换按钮，即可来回开启和关闭；

1. 关闭状态：
![](/conalog-doc/styles/images/stop.png)
2. 开启状态：
![](/conalog-doc/styles/images/start.png)

  6.1.2 查看
1. 执行成功：  
Exec Count: 执行次数；  
Last Activity Time: 最后一次执行时间；  
Last Activity Message: 最后一次执行信息；  
&nbsp; &nbsp; &nbsp;stdout: 执行过程中输出的正确数据；  
&nbsp; &nbsp; &nbsp;stderr:执行过程中发现的错误；  
![](/conalog-doc/styles/images/statusMessage.png)
2. 执行失败：  
Exec Count: 0；  
Last Activity Time: N/A；  
Last Activity Message: N/A / Pending；  
![](/conalog-doc/styles/images/start.png)
3. 查看redis通道数据:  
打开终端；  
输入 redis-cli；  
subscribe redis channel (active collector即为 ac_[collector name], passive collector 即为 pc _[collector name]);

  6.2 parser status
---------------------

 6.2.1 启动parser实例
1. 点击start按钮：
![](/conalog-doc/styles/images/startParserInstance.png)
2. 弹出确认框,点击确定即生成实例，点击取消即取消生成：
![](/conalog-doc/styles/images/startParserInstanceModal.png)


 6.2.2 查看parser实例
1. 查看实例总数：
![](/conalog-doc/styles/images/instanceNum.png)
2. 查看实例内容，点击下拉按钮,即可显示所有实例：
![](/conalog-doc/styles/images/showParserInstance.png)
![](/conalog-doc/styles/images/parserInstanceDetail.png)
3. 查看redis通道数据:  
打开终端；  
输入 redis-cli；  
subscribe redis channel (parser outputChannel);  


 6.2.3 删除parser实例
1. 点击stop按钮；
![](/conalog-doc/styles/images/deleteParserInstance.png)
2. 弹出确认框,点击确认即删除，点击取消即取消删除：
![](/conalog-doc/styles/images/stopParserInstanceModal.png)


第七章 history
====================================

history作用：保存日志，默认数据保存时限为7天。

  7.1 查询
---------------------

1. 根据EventID进行查询：
![](/conalog-doc/styles/images/history.png)
2. 点击下拉按钮，查看具体内容：
![](/conalog-doc/styles/images/showHistory.png)
![](/conalog-doc/styles/images/historyContent.png)


第八章 常用流程
====================================


  8.1 数据采集流程
---------------------

1. 添加cert；
2. 添加 collector；
3. status里启动并查看结果；

  8.2 数据采集-解析流程
---------------------

1. 添加cert；
2. 添加collector, 根据需求选择不同的collector类型；  
Active collector 输出数据的redis通道名为 ac\_[collector name]；   
Passive collector 输出数据的redis通道名为 pc\_[collector name]；   
Agent collector 输出数据的redis通道名为 agt_[collector name]；
3. 添加parser，InputChannel为collector数据输出通道名，collector采集的数据则可以进入Parser；
4. status里启动并查看结果；

  8.3 外部脚本监控流程
---------------------

1. 添加cert；
2. 添加 Active collector，定时启动外部脚本；
3. status里启动并查看结果；

  8.4 状态监控流程
---------------------

1. 采集Orientsoft内部软件的日志;
3. history进行展示；











