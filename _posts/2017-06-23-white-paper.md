---
layout: post
title:  white-paper
date:   2017-06-23 00:00:00 +0800
categories: document
tag: 白皮书
---

* content
{:toc}


第一章 问题和思路  
=========

1.1 数据采集、解析的开发难度问题 
------------------------------

   ### 数据采集的难度  
      尽量减小对目标系统的干扰  
      平衡采集频率和吞吐量  
      稳定可靠的通信  
   ### 解析的难度  
      对数据中的错误具有鲁棒性  
      对于复杂数据的处理能力  
      能够处理嵌套、混合类型的数据  
      对数据采集速度的变化具有缓冲、容纳能力  
   ### 解决思路
      基于远程命令执行和网络文件映射的数据采集  
      在架构方面，将采集器进行抽象（主动、被动）  
      提供解析器模板、框架  
      提供多种通信信道（缓冲、分流、错误信道）  
      
   1.2 数据采集、解析等自编制脚本的管理问题  
   -------------------------------------
   
   ### 目前自编制脚本的问题  
      缺乏执行状态监控，可靠性难以保证  
      缺乏知识、代码积累  
      缺乏文档支持  
      缺乏配置、部署的管理  
   ### 解决思路  
      提供状态监控  
      在状态监控的基础上，提供异常状态重启  
      强制要求文档的代码库  
      提供配置部署选项  
1.3 Orientsoft软件组件状态管控的问题  
----------------------------------

   ### 目前Orientsoft软件组件状态的问题  
      缺乏状态监控  
      缺乏有效的日志  
   ### 解决思路  
      利用conalog ActiveCollector采集组件状态（组件进程的存活、对系统资源的占用）  
      提供日志发布API，由conalog统一管理日志（Rotate、查询、导出）  


第二章 最佳实践  
==============

2.1 数据采集、解析脚本  
--------------------

   ### 数据采集  
     对于数据采集脚本，推荐尽量以重复短暂执行模式为主，这类脚本用ActiveCollector来调用  
     获取到的数据不用开Redis通道，直接打印到STDOUT上，并且不用手工分行；如果有错误，直接输出到STDERR上。采集完成进程直接退出  
     需要长期实时采集的数据，推荐用SSHFS映射到本地后，用Filebeat采集到Redis通道上，在Conalog中添加AgentCollector，利用正则表达式过滤出需要的文件名/路径，并发送到指定通道  
   ### 解析脚本  
     解析脚本内部一般需要实现某种形式的Buffer，用来应对数据行的切断和数据输入速度的变化  
     同时解析脚本需要特别注意错误状态的恢复，如果陷入错误应该设法回到初始状态  
     
2.2 脚本管理  
------------

   ### 知识积累  
     项目部署完成，及时上传新脚本到Git  
     添加Collector/Parser的时候尽量编写完善的备注，建议给出数据/结果样本  

2.3 日志管控  
-----------

   ### 日志集中  
     推荐使用Conalog提供的Restful API接口直接将日志推送到Conalog  
     编写统一的脚本进行资源状况采集  