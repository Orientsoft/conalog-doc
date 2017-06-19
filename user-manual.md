## 第一章 概述
conalog是集数据采集，解析，状态管理，日志管理为一体的工具，可以有效的对自编制脚本进行管理和监控，也可以监控orientsoft软件组件的状态,主要模块分为五部分：cert，collector，parser，status，history。
## 第二章 部署与安装
### 2.1 环境要求
Linux版本：<br>
Node版本：6.9 (LTS)<br>
Mongo版本：<br>
Redis版本：<br>
### 2.2 前台
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

### 2.3 后台
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

### 2.4 常见问题
1. 每次修改配置文件后要运行 gulp go 使修改生效;
2. 执行gulp install报错 "无法找到gulp" 时即执行 'npm install -g gulp'；
3. 运行npm i命令后遇到缺少模块的问题时，npm install 该模块；

### 2.5 访问网址
conalogHost:conalogFrontPort<br>
如：'192.168.0.244:7527'<br>

登录界面：（账号：admin 密码：admininitpass）
![](images/logIn.png)

登录成功：
![](images/homePage.png)

## 第三章 cert
cert功能：通过ssh连接登陆虚拟机，随后可以执行Shell命令。
### 3.1 添加
1. 点击左上角添加按钮：
![](images/addCert.png)
2. 弹出添加框，填写信息：</br>
   填写规范：</br>
   Host: 192.168.0.244，（虚拟机IP地址）</br>
   Port：22，（端口号）</br>
   User：voyager，（虚拟机用户名）</br>
   Password：welcome1，（虚拟机用户名对应的密码）
![](images/addCertModal.png)

3. 添加成功：
![](images/addCertSuccess.png)

### 3.2 修改
1. 点击edit按钮：
![](images/editCert.png)
2. 弹出修改框，修改信息：
![](images/editCertModal.png)
3. 保存即点击确认，不保存即点击取消；

### 3.3 删除
1. 点击delete按钮： 
![](images/deleteCert.png)
2. 弹出确定框，点击确认即删除，点击取消即取消删除：
![](images/deleteCertModal.png)

### 3.4 查看密码
1. 鼠标放在图标上，password即显示为明文密码，鼠标移开，密码即为隐藏密码；
![](images/checkPassword.png)


## 第四章 collector
collector作用：实时执行命令，采集数据，分为active collector，passive collector和agent collector，active collector是根据设定的时间间隔执行一次命令，passive collector是执行一次命令并一直保持执行状态，agent collector是在Filebeat把监听的所有日志更新发送到一个统一的通道后，根据通配符规则，把同类型的日志，分发到一个通道中。
### 4.1 添加
1. active collector：</br>
填写规范：</br>
Name: 无要求（输出数据的redis通道名默认为 ac_name);</br>
Type:</br>
&nbsp; &nbsp; &nbsp; &nbsp;interval : 每间隔一段时间执行一次命令；</br>
&nbsp; &nbsp; &nbsp; &nbsp;time ：每天定点执行命令；</br>
&nbsp; &nbsp; &nbsp; &nbsp;oneshot：只执行一次；</br>
Trigger: 对应type类型设置时间；</br>
Command: 执行命令；</br>
Parameter: 参数；</br>
Host: 虚拟机IP；</br>
Encoding: 根据电脑系统选择对应的编码；</br>
Channel: redis/nanomsg;</br>
Description: collector usage & source & description；
![](images/addActiveCollector.png)
2. passive collector：</br>
填写规范：</br>
Name: 无要求（输出数据的redis通道名默认为 pc_name);</br>
Type:</br>
&nbsp; &nbsp; &nbsp; &nbsp;LongScript: 执行用户指定的命令</br>
&nbsp; &nbsp; &nbsp; &nbsp;File Tail: 直接执行 tail -F 命令</br>
Command: 执行命令；</br>
Parameter: 参数；</br>
Host: 虚拟机IP；</br>
Encoding: 根据电脑系统选择对应的编码；</br>
Channel: redis/nanomsg;</br>
Description: collector usage&source&description；
![](images/addPassiveCollector.png)
3. agent collector<br>
&nbsp;3.1 点击添加按钮：
![](images/addAgentCollector.png)
&nbsp;3.2 弹出添加框，填写信息：</br>
&nbsp;&nbsp;&nbsp;填写规范</br>
&nbsp;&nbsp;&nbsp;Name: 无要求；</br>
&nbsp;&nbsp;&nbsp;Parameter: 文件名的正则表达式；</br>
&nbsp;&nbsp;&nbsp;Encoding: 根据电脑系统选择对应的编码；</br>
&nbsp;&nbsp;&nbsp;Channel: redis/nanomsg;</br>
&nbsp;&nbsp;&nbsp;Description: collector usage & source & description；
![](images/addAgentCollectorContent.png)

### 4.2 修改
1. active collector：</br>
   勾选要修改的项，再点击edit按钮，在页面上方即可出现对应的信息，修改之后保存点击save按钮，不保存点击clear按钮；
![](images/editActiveCollector.png)
![](images/editActiveCollectorContent.png)
2. passive collector：</br>
   勾选要修改的项，再点击edit按钮，在页面上方即可出现对应的信息，修改之后保存点击save按钮，不保存点击clear按钮；
![](images/editPassiveCollector.png)
![](images/editPassiveCollectorContent.png)
3. agent collector:</br>
   点击edit按钮，即会弹出修改框，修改之后保存点击确定按钮，不保存点击取消按钮；
![](images/editAgentCollector.png)
![](images/editAgentCollectorContent.png)


### 4.3 删除
1. active collector：</br>
   勾选要删除的项，再点击delete按钮，弹出确定框，删除点击确定，不删除点击取消；
![](images/editActiveCollector.png)
![](images/deleteActiveCollector.png)
2. passive collector：</br>
   勾选要删除的项，再点击delete按钮，弹出确定框，删除点击确定，不删除点击取消；
![](images/editPassiveCollector.png)
![](images/deletePassiveCollector.png)
3. agent collector:</br>
   点击delete按钮，弹出确定框，删除点击确定，不删除点击取消；
![](images/deleteAgentCollector.png)
![](images/deleteAgentCollectorModal.png)


## 第五章 parser
parser的功能：parser通过调用脚本把文件中的文本数据转换成结构数据，同一个parser可以同时启动多个实例。
### 5.1 添加
1. 点击左上角添加按钮：
![](images/addParser.png)
2. 弹出添加框，填写内容，所有选项均为必填：</br>
填写规范：</br>
Name：esb   (无要求)；</br>
Path：esb.js   (parser脚本的路径)；</br>
Parameter：esb=1   (脚本对应的参数)；</br>
InputChannel：ac\_mobile   (输入数据通道名)；</br>
OutputChannel：p_esb   (输出数据通道名)；</br>
InputType：RedisChannel   (RedisChannel/NanomsgQueue);</br>
OutputType：RedisChannel   (RedisChannel/NanomsgQueue);</br>
Remark：input:... output:{...}   (parser脚本作用描述，输入输出数据格式等);
![](images/addParserContent.png)
3. 添加成功：
![](images/addParserSuccess.png)

### 5.2 修改
1. 点击edit按钮：
![](images/editParser.png)
2. 弹出修改框，修改信息：
![](images/editParserModal.png)
3. 保存即点击确认，不保存即点击取消；

### 5.3 删除
1. 点击delete按钮： 
![](images/deleteParser.png)
2. 弹出确定框，点击确认即删除，点击取消即取消删除：
![](images/deleteParserModal.png)


## 第六章 status
status作用：展示active collector，passive collector，agent collector以及parser执行的状态和结果。
### 6.1 active ／passive／agent status
#### 6.1.1 控制
点击operation下的切换按钮，即可来回开启和关闭；

1. 关闭状态：
![](images/stop.png)
2. 开启状态：
![](images/start.png)

#### 6.1.2 查看
1. 执行成功：</br>
Exec Count: 执行次数；</br>
Last Activity Time: 最后一次执行时间；</br>
Last Activity Message: 最后一次执行信息；</br>
&nbsp; &nbsp; &nbsp;stdout: 执行过程中输出的正确数据；</br>
&nbsp; &nbsp; &nbsp;stderr:执行过程中发现的错误；</br>
![](images/statusMessage.png)
2. 执行失败：</br>
Exec Count: 0；</br>
Last Activity Time: N/A；</br>
Last Activity Message: N/A / Pending；</br>
![](images/start.png)
3. 查看redis通道数据:</br>
打开终端；</br>
输入 redis-cli；</br>
subscribe redis channel (active collector即为 ac_[collector name], passive collector 即为 pc _[collector name]);

### 6.2 parser status
#### 6.2.1 启动parser实例
1. 点击start按钮：
![](images/startParserInstance.png)
2. 弹出确认框,点击确定即生成实例，点击取消即取消生成：
![](images/startParserInstanceModal.png)


#### 6.2.2 查看parser实例
1. 查看实例总数：
![](images/instanceNum.png)
2. 查看实例内容，点击下拉按钮,即可显示所有实例：
![](images/showParserInstance.png)
![](images/parserInstanceDetail.png)
3. 查看redis通道数据:</br>
打开终端；</br>
输入 redis-cli；</br>
subscribe redis channel (parser outputChannel);</br>

```
//正确的输出格式：
{

}
```


#### 6.2.3 删除parser实例
1. 点击stop按钮；
![](images/deleteParserInstance.png)
2. 弹出确认框,点击确认即删除，点击取消即取消删除：
![](images/stopParserInstanceModal.png)



## 第七章 history
history作用：保存日志，数据保存时限为7天。
### 7.1 查询
1. 根据EventID进行查询：
![](images/history.png)
2. 点击下拉按钮，查看具体内容：
![](images/showHistory.png)
![](images/historyContent.png)


## 第八章 常用流程

### 8.1 数据采集流程
1. 添加cert；
2. 添加 collector；
3. status里启动并查看结果；

### 8.2 数据采集-解析流程
1. 添加cert；
2. 添加collector, 根据需求选择不同的collector类型；<br>
Active collector 输出数据的redis通道名为 ac\_[collector name]； <br>
Passive collector 输出数据的redis通道名为 pc\_[collector name]； <br>
Agent collector 输出数据的redis通道名为 agt_[collector name]；
3. 添加parser，InputChannel为collector数据输出通道，collector采集的数据则可以进入Parser；
4. status里启动并查看结果；

### 8.3 外部脚本监控流程
1. 添加cert；
2. 添加 Active collector，定时启动外部脚本；
3. status里启动并查看结果；

### 8.4 状态监控流程











