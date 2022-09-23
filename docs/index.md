# **手摸手Prometheus&Thanos监控技术与实战教程**

> Started at Oct 16, 2018 By Jacob Xi
> 
> Prometheus, Grafana, AlertManager, PromQL, Thanos, VictoriaMetrics and APM

## **JX's chit-chat**

> Hi, this part is fucking new and freshing🤔

记录一下生活，大概是上周四19号的时候终于拿到了正式的offer letter, associate director 副董事，也是我这辈子第一次有了自己的title，听起来还蛮唬人的。不过应该还是一个悲催的打工仔，既然已经拿到了属于自己奋斗的结果，终于可以按部就班的实现自己的计划了，希望一切顺利。这周一人生第一次主动提离职，有点小紧张，满肚子的怨气还是没有撒出去，不过我已经peace了。 

**感觉快要解封了，希望吧，不要再欺骗我们这些单纯的小伙子们了🤠**

> 说说最近看的新剧和小说吧

* “神探波罗”马上就要看完了，虽然感觉有点疲了，但是不得不说，真的还是真精致的剧，老戏骨的演技没得说
* 东京罪恶 Tokyo Vice, 好久没有看到节奏和画面这么棒的电视剧了，看完真的好想去东京转一圈
* 史前星球 Prehistoric Planet 不多说，每一帧都可以截图作为壁纸，以白垩纪时代的环境为背景，描绘了海岸、沙漠、河流、冰雪世界和森林的场景，展示了鲜为人知且令人惊讶的恐龙生活。
* 小说，最近看完了《寄生前夜》，《霍比特人》，觉得自己年纪大了，反而开始喜欢这种慢节奏的中世纪小说了

## 内容简介

本书是个特别有意思的一次尝试，第一篇文章发表于2018年10月，陆陆续续写了四年多，90多篇文章，从BB到SAP，甚至马上就要到新公司了。突然发现随着这些孤立的文章越写越多，不如把这些文章整理整理出一本电子书，有关Prometheus监控技术栈，包括Prometheus的使用，Prometheus stack的搭建，服务的抓取，PromQL 的使用，好用以及常用的Alert信息， Prometheus HA集群 Thanos，Prometheus 持久化集群 VictoriaMetrics，以及APM监控的介绍

* Promethues应用监控介绍，K8S 里面暴露信息常用的kube-state-metrics，HPA,  Metrics Server, 传统的安装方式
* Promethues on Linux, Node Exporter，Prometheus MySQL 可视视化监控
* 手动部署Kubernetes Prometheus， 玩转Prometheus Operator部署，Prometheus 服务的自动发现使用
* Prometheus-Adapter 实现 应用进行自定义指标扩缩容
* PromQL的学习包括，操作符 & 内置函数，简单示例，记录规则的使用(Recording Rule)
* Awesome Prometheus alerts 
* 数据可视化 Grafana 以及几个常用的 Grafana Plugin DevOpsProdigy KubeGraf， darks
* 大规模场景下 Prometheus 的优化手段 Thanos 
* Prometheus高性能远程存储VictoriaMetrics
* 分布式链路追踪技术 以及 APM的应用介绍

![Alt Image Text](./images/bg1_1.png "body image")

![Alt Image Text](./images/bg1_2.png "body image")

![Alt Image Text](./images/bg1_3.png "body image")

### **Previous on my Technolog book**

> [手摸手 Jenkins 战术教程 (大师版）](https://chao-xi.github.io/jxjenkinsbook/)
> 
> [手摸手 Elasticsearch7 技术与实战教程](https://chao-xi.github.io/jxes7book/)
> 
> [手摸手 Redis 技术与实战教程](https://chao-xi.github.io/jxredisbook/)
> 
> [手摸手 Chef & Ansible 技术与实战教程](https://chao-xi.github.io/jxchefbook/)
> 
> [手摸手 分布式与流式系统 (In progress)](https://chao-xi.github.io/jxdmsbook/)
> 
> [Azure 103&900 Tutorial (In progress)](https://chao-xi.github.io/jxazurebook/)
> 
> [手摸手 Linux Performance & 面试实战教程](https://chao-xi.github.io/jxperfbook/)
>
> [手摸手 Databases 全教程](https://chao-xi.github.io/jxdatabasebook/)
> 
>  [AWS Certified Data Analytics Tutorial](https://chao-xi.github.io/jxawscbdbook/)
> 
> [Istio & Service Mesh 战术教程](https://chao-xi.github.io/jxistiobook/)
> 
> [AWS Certification Solutions Architect Book](https://chao-xi.github.io/jxawscsaabook/)
> 
> [手摸手Prometheus&Thanos监控技术与实战教程](https://chao-xi.github.io/jxprombook/)
> 
> [Distributed Message System Book(Kafka)](https://chao-xi.github.io/jxdmsbook/)

## **Salut! C'est Moi**

> Do or Do not, there is no try

Hello, this is me, Jacob. Currently, I'm working as DevOps and Cloud Engineer in SAP, and I'm the certified AWS Solution Architect and Certified Azure Administrator, Kubernetes Specialist, Jenkins CI/CD and ElasticStack enthusiast. 

I was working as Backend Engineer in New York City and achieved my CS master degree in SIT, America. Believe it or not, I'll keep writing, more and more books will come out at such dramatic and unprecedented 2021. 

If you have anything want to talk to me directly, you can reach out for via email xichao2015@outlook.com。


Salute, c'est moi, Jacob. Actuellement, je travaille en tant qu'ingénieur DevOps et Cloud dans SAP, et je suis architecte de solution AWS certifié et administrateur Azure certifié, spécialiste Kubernetes et passionné de CI/CD.

Je travaillais en tant qu'ingénieur backend à New York et j'ai obtenu mon master CS à SIT, en Amérique. Croyez-le ou non, je continuerai à écrire, de plus en plus de livres sortiront cette année.

![Alt Image Text](./images/bg1_2.png "body image")

## **To be continue**

I will start working on Fintech book later on and in future, I will put more effort do finish "Distributed Message System Book".🙂

![Alt Image Text](./images/bg1_4.png "body image")