### 起因

个人判断国内区块链发展方向应为联盟链，这在[区块链技术总结及发展展望](https://www.jianshu.com/p/3a8b6cf9f6d3）)一文中已详细阐述。并在主导的联盟链项目中选择**Hyperledger Fabric**作为底层技术框架（R3的Corda需要收费其主要针对金融项目，金链盟主导的FISCO BCOS还处于发展之中）。

**Hyperledger Fabric**入门并不复杂，按照官方例子[Building Your First Network](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html)很容易完成链搭建，创建通道，部署合约，调用合约等过程。

官方例子虽然容易上手，但不适合直接应用到实际项目中

- 所有的资源和脚本都在一起，并没有按照组织分开。实际项目中每个组织只会拥有自己相关的资源和脚本。
- 证书集成生成。实际项目中每个组织管理各自的证书，必须确保自己证书的安全。
- 证书信息，节点名称，通道名称等为示例信息，需要根据项目修改。
- 缺少SDK调用示例。

但当我们开始根据项目搭建自定义的联盟链时，却遇到了很多问题，前后花了不少时间，直到基本搞清例子中每个参数含义后，才完成搭建。特编写此教程实现一个完整可运行的定制化demo，记录整个过程，也希望帮助有同样需要的同学少走弯路。

### 介绍

本教程主要展示定制化联盟链搭建过程，默认阅读者已熟悉**Hyperledger Fabric**相关基本概念。如果您不熟悉，请先参阅[官方文档](https://hyperledger-fabric.readthedocs.io/en/release-1.4/key_concepts.html)，英文不好者可以参阅[社区中文版](https://hyperledgercn.github.io/hyperledgerDocs/)。**社区中文版时效性不是很好，强烈建议阅读英文原版，一切以官方文档为准。**

本教程示例**MyFabric Demo**采用**Hyperledger Fabric v1.4**（Fabric第一个LTS版本），主要参考 [Building Your First Network](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html)和 [fabric-ca](https://github.com/hyperledger/fabric-samples/tree/release-1.3/fabric-ca)。

### 特点

- 按照组织管理资源和脚本
- 采用两级fabric-ca-server生成证书
- 修改证书信息，节点名称，通道名称等信息
- 提供Java SDK调用示例。

### 大纲

1. [MyFabric Demo链组织及身份生成](https://www.jianshu.com/p/ffcf6d5dfdaf)
2. [MyFabric Demo链节点启动及合约部署调用](https://www.jianshu.com/p/e6cf9df5fcc3)
3. [MyFabric Demo Java SDK访问示例](https://www.jianshu.com/p/1fce62691d4c)

### 源代码
本教程的完整代码可以从**GitHub**上下载
**MyFabric链代码** - [myfabric-demo-chain](https://github.com/fftt2017/myfabric-demo-chain)
**MyFabric Java客户端代码** - [myfabric-demo-java-client](https://github.com/fftt2017/myfabric-demo-java-client)






