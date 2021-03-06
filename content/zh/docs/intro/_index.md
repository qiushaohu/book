
---
title: "夜莺介绍"
linkTitle: "夜莺介绍"
weight: 2
date: 2020-02-29
description: >
  对夜莺开源社区和产品有个总体的介绍，附联系方式，欢迎社区朋友跟我们一起共建生态。
---

## 欢迎加入Nightingale社区

Nightingale是一套衍生自Open-Falcon的互联网监控解决方案，融入了滴滴的最佳实践，由于改动太大，优化太多，产品上已经无法与Open-Falcon平滑兼容，故而单开一个项目。

编写Open-Falcon第一行代码是在2014年，几年下来，有上千家商业公司在生产环境使用，我们也在逐步优化，夜莺可以说是一个颠覆级版本，性能、易用性、可用性都做了大幅改进，在滴滴抗住了7.7亿（资源、组件、应用、业务各层次的监控都有）监控指标，这是一个新的征程，期待与诸君携手，把监控这个领域，做到极致！

## 开源代码

**[💡💡💡💡💡 https://github.com/didi/nightingale (来个star支持下呀~) 💡💡💡💡💡](https://github.com/didi/nightingale)**

## 系统截图

![Nightingale](https://s3-gz01.didistatic.com/n9e-pub/image/snapshot/n9e-screen.png)

上图是监控告警子系统的监控大盘页面，夜莺已经不只是一个运维监控系统，还有一部分CMDB的能力、自动化运维的能力，很多公司都基于夜莺开发自己公司的运维平台。

## 各子系统简介

夜莺拆成了四个子系统，分别是：

1. **用户资源中心（RDB）**
2. **资产管理系统（AMS）**
3. **任务执行中心（JOB）**
4. **监控告警系统（MON）**

下面分别介绍一下这几个子系统的设计初衷。

### 用户资源中心

这是一个平台底座，所有的运维系统，都需要依赖此系统，内置用户、权限、角色、组织、资源的管理。最核心的是一棵组织资源树，树节点的类别和扩展字段可以自定义，组织资源树的层级结构最简单的组织方式是：`租户>项目>模块`，复杂一点的组织方式：`租户>组织>项目>模块>集群`，组织是可以嵌套的。
    
节点上挂两类对象，一个是人员权限，一个是资源，资源可以是各类资源，除了主机设备、网络设备，也可以是`rds`实例，`redis`实例，当然，这就需要`rds`、`redis`的管控系统和**RDB**打通了。滴滴在做一些大的中后台商业化解决方案的时候，**RDB**就是扮演了这么一个底座的角色。

### 资产管理系统

这里的资产管理系统，是偏硬件资产的管理，这个系统的使用者一般是系统部的人，资产管理类人员，应用运维相对不太关注这个系统。开源版本开放了一个主机设备的管理，大家可以二开，增加一些网络设备管理、机柜机架位的管理、配件耗材的管理等等，有了底座，上面再长出一些其他系统都相对容易。
    
agent安装完成之后，会自动注册到资产管理系统，自动采集到机器的 `sn`、`ip`、`cpu`、`mem`、`disk` 等信息，这些信息为了灵活性考虑，都是用 `shell` 采集的，上文安装步骤一章有提到，其中最重要的是 `ip`，系统中有很多设备，`ip` 是需要全局唯一，其他的 `sn`、`ip`、`cpu`、`mem`、`disk` 等，如果无法采集成功，可以写为固定值，`shell` 里直接写 `echo` 一个假数据即可。

每一条资产，都有一个租户的字段，代表资产归属，需要管理员去分配资产归属（修改资产的所属租户），各个租户才能使用对应的资产，分配完了之后，会出现在用户资源中心的"游离资源"菜单中，各个租户就可以把游离资源挂到资产树上去分门别类的管理使用。树节点的创建是在树上右键哈。

### 任务执行中心

用于批量跑脚本，类似**pssh**、**ansible**、**saltstack**，不过不支持playbook，大道至简，就用脚本撸吧，`shell`、`python`、`perl`、`ruby`，都行，只要机器上有解析器。因为是内置到夜莺里的，所以体系化会更好一些，和组织资源树的权限是打通的，可以控制不同的人对不同的机器有不同的权限，有些人可以用`root`账号执行，有些人只能用普通账号执行，历史执行记录都可以通过web页面查看审计。任务本身支持一些控制：**暂停点**、**容忍度**、**单机超时时间**、**中途暂停**、**中途取消**、**中途Kill**等。

一些经常要跑的脚本，可以做成模板，模板是对脚本的一种管理方式，后续就可以基于模板创建任务，填个机器列表就可以执行。比如**安装JDK**，**调整TCP内核参数**，**调整ulimit**等机器初始化脚本，都可以做成模板。

开源版本的任务执行中心，可以看做是一个命令通道，后续可以基于这个命令通道构建一些场景化应用，比如**机器初始化平台**、**服务变更发布平台**、**配置分发系统**等。任务执行中心各类操作都有 `API` 对外暴露，具体可参看：[router.go](https://github.com/didi/nightingale/blob/master/src/modules/job/http/router.go) 我司的命令通道每周执行任务量超过60万，就是因为各类上层业务都在依赖这个命令通道的能力。

### 监控告警系统

这块核心逻辑和**v2**版本差别不大，监控指标分成了设备相关指标和设备无关指标，因为有些自定义监控数据的场景，`endpoint`不好定义，或者 `endpoint` 经常变化，这种就可以使用设备无关指标的方式来处理。监控大盘做了优化，引入了更多类型的图表，但夜莺毕竟是个 `metrics` 监控系统，处理的是数值型时序数据，所以，最有用的图表其实就是折线图，其他类型图表，看看就好，场景较少。夜莺也可以对接Grafana，有个专门的 [DataSource插件](https://github.com/n9e/grafana-n9e-datasource)，Grafana会更炫酷一些，只是，在数据量大的时候性能较差。

## 系统架构

![n9e系统架构图](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-v3-arch.png)

当前是v3架构，请大家先看视频学习[v2的架构](https://www.bilibili.com/video/BV1K54y1R7wS/)，然后再来看这张架构图，监控部分的架构和之前没有差别，**collector**揉进了一些命令执行的能力，所以改了个名字叫**agent**。引入了三个新组件：**rdb**、**ams**、**job**，**rdb**是用户资源中心，**ams**是资产管理系统，**job**是任务执行中心。**agent**除了上报监控数据给**transfer**，还会上报本机信息给**ams**，注册本机信息到资产管理系统，另外就是与**job**模块交互，拉取要执行的任务，上报任务执行结果。

## 联系我们

夜莺是Obsuite的一个子品牌，Obsuite官方公众号，夜莺相关活动、教程、资料都可以在公众号找到

<img src="https://s3-gz01.didistatic.com/n9e-pub/image/obsuite/obsuite-wx.jpg" width="200" alt="obsuite-wx"/>


