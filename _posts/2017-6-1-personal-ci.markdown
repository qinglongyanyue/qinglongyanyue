---
layout: default
title:  "云研发思考——粗犷型研发的问题"
date:   2017-06-01
categories: devops
---

作为一个云时代的开发者，云能给开发者带来哪些有价值的变化了？云能成为开发者工具箱中的一种工具么?

我司大多数程序猿的标配一般是一台Windows桌面用于写代码，一台Linux VM用于编译调试代码，有钱的主可能直接配一台物理机。做分布式软件的，一般每个人都有3台VM。

而且咱们都觉着咱们的研发效率挺高的啊，都这么玩了这么多年，一切都挺习惯的啊。云是啥？是VM么，我们已经在用了啊，除了方便安装OS，也没发现有多方便呢？

深入分析下，会发现这种研发方式有3个问题，我们取个名字叫粗犷型研发：
- 效率问题
- 成本问题
- 团队协作问题

## 效率问题
- 申请机器安装OS，配置代理，配置软件环境一般都需要花费大量的时间，每个人都来一遍存在较大浪费
- 一般配置好一个环境就不太愿意改动，配置好一个靠谱的环境太难了，坚决不能动
- 如果支撑新的OS，新的中间件，需要申请新的VM，再配置新的环境；最终每个人都会有多个不同OS的VM，费时间，费金钱
- hicloud经常会出现资源不足，需要很长时间才能申请到资源，尽管其他人手中存在大量闲置的VM
- 多个VM和Windows桌面之间代码人肉拖来拖去，效率低，容易出错
- 整个静态检查，编译，LLT等动作需要手工动作，比如敲./configure，make, make test，make install, start.sh等。
- windows写代码，linux编译测试，也需要多次切换屏幕，切换OS，应该还有同学跟我一样，偶尔会在VIM中敲ctrl + S保存修改。
- 还有很多研发的过程数据都在我们手工操作中流逝了，其实很多数据都有很有价值的，比如测试覆盖率统计，编译成功率，静态检查过程数据。收集这些数据是有利于个人去改进工作效率的。

## 成本问题
成本问题我们是深有体会的：hicloud的VM账单到来时，我们才发现，部门200多人就有1500多个VM的用量，人均7.5台，每台VM均价约3000元一年，总计约450万一年。这还不算物理服务器开销，预计也是每年几百万的量级。为什么一个200多人的团队会需要这么多的研发资源呢？什么原因导致这种情况呢？

- 研发个人需要多套环境，分布式软件的就更多了；而且大部分情况下利用率并不高，绝大多数时候都处于空转费电状态，但是又没有好的渠道共享给他人使用。
- 研发的个人桌面云也是类似，不用的时候一样在后台费着电呢
- CI团队需要项目级和版本级的构建环境，一般规模都会比较大，但是实际只是每天定时构建几次而已，99%的时间在空转
- 测试人员又需要N套独立的环境进行各种测试，利用率同样不高
- 如果有团队部署演示环境，那么演示环境又是一批独立的VM
- 如果有团队把软件run起来对外提供服务，那么又是一批新的VM
- 为了避免需要的时候没有资源可用，于是乎每个项目团队都会囤积大量的资源；早期我们team就是怕到处借设备，10人团队居然囤积了50多台服务器，几百块硬盘，各种交换机。反正我们世界500强，不缺钱。

## 团队协作问题

- 每个人都都不同的工具配置逻辑，相互之间配合的时候存在一定的麻烦，比如shell的配置，git的配置，VIM的配置等
- 有时候代码在A同学的环境一切都是OK的，但是切换到B同学的机器就是不行，遇到的概率还不低；因为每个人的环境都有自己的DIY在里面，并不是统一的clone
- 还有些同学喜欢写一些自己玩的脚本，比如configure.sh,make.sh,start.sh很多时候其他的同学想用就会比较纠结，需要DIY的时候就在自己的环境搞个私有版本，结果人人都有一个私有版本的脚本，脚本演进难以完成。而在云中，这些自动化的能力是核心，马虎不得。
- 其实研发的动作是完全类似的，不外乎写代码，静态检查，编译，LLT，出包；服务化的团队会设计部署，黑盒测试，发布升级。无论是个人级过程，项目级过程，版本级过程，本质是类似的，只是代码涉及的规模不同，检查的细致程度不同而已。粗犷型的模式并没有一种思考将个人的这些重复性动作抽象成一种服务？

去年读到Google的论文，“Large-scale cluster management at Google with Borg”，其中明确的提到Google的开发，测试，生产，数据分析都是在一套环境中完成，一切都是不同task而已：
- Google搜索，地图等是长期运行的task，需要确保长期稳定可靠运行，优先级最高
- 编译、测试，静态检查之类的都是临时task，跑完一遍就销毁，需要快速响应，优先级较高
- 数据分析类的task，定期临时获取一些监控数据，并进行分析，不要求实时，优先级较低
- 内部使用容器技术隔离各个task，为每个task配置足够的CPU，MEM，网络，存储资源。
- 使用BORG系统进行整个任务的调度，每个任务在哪里run没人知道，反正集群的任何位置都有可能，看哪里空闲就去哪里。

说实话看了这篇论文之后，很受震撼，生产环境居然和测试开发环境在一起，不可思议啊，但是人家就是做到了，赶紧学习吧。。小米加步枪如何赶得上航母？一天干24小时也怕是不够啊。。

下篇我会就介绍下公司内部好多有理想的青年为云研发做的初步工作，期待咱们研发云化，精细化的理想早日实现。
