本文为[Hyperledger Fabric定制化联盟链搭建教程](https://www.jianshu.com/p/4fd103dee864)系列第一篇，介绍MyFabric Demo链的各种组织和身份如何创建。

本篇代码可以从Github上下载 - [myfabric-demo-chain](https://github.com/fftt2017/myfabric-demo-chain)

### 代码目录结构如下

```
myfabric-demo-chain
├── chaincode
├── org0                  //org0组织目录
├── org0-orderer
├── org1                  //org1组织目录
├── org1-peer
├── org2                  //org2组织目录
├── org2-peer
├── peer-cli
├── scripts
├── start-ca.sh           //ca启动脚本
├── start-chain.sh
├── start.sh
├── stop-ca.sh            //ca结束脚本
├── stop-chain.sh
└── stop.sh
```

本节教程相关部分已用`//`标出。

### MyFabric Demo链包含3个组织，org0，org1和org2

`start-ca.sh`负责调用3个组织里面的`start.sh`脚本

`stop-ca.sh`负责调用3个组织里面的`stop.sh`脚本

#### org0提供1个orderer节点，目录结构如下

```
org0
├── config
│   ├── ica-fabric-ca-server-config.yaml      //中间CA服务配置文件
│   └── rca-fabric-ca-server-config.yaml      //根CA服务配置文件
├── data
│   ├── admin                                                      
│   │   ├── msp                               //组织admin msp目录
│   │   └── tls                               //组织admin tls目录
│   ├── msp                                   //组织msp目录
│   │   ├── admincerts
│   │   ├── cacerts
│   │   ├── intermediatecerts
│   │   ├── keystore
│   │   ├── signcerts
│   │   ├── tlscacerts
│   │   ├── tlsintermediatecerts
│   │   └── user
│   ├── orderer                                                    
│   │   ├── msp                               //组织orderer msp目录
│   │   └── tls                               //组织orderer tls目录
│   ├── org0-ca-cert.pem                      //组织根CA证书
│   └── org0-ca-chain.pem                     //组织中间CA证书
├── docker-compose.yml
├── start.sh
└── stop.sh
```

- `start.sh`脚本启动docker compose，生成`data`下的各个身份目录

- `stop.sh`脚本关闭docker compose

- `docker-compose`启动3个docker，根CA服务docker，中间CA服务docker，身份生成docker

**Hyperledger Fabric**通过**MSP**管理各种身份，详细概念请参阅[Membership](https://hyperledger-fabric.readthedocs.io/en/release-1.4/membership/membership.html)。

主要有两个用途

1. **channel msp** - 保存在链上，用来定义组织管理员，CA证书等信息。此msp没有自己的身份文件。上图中`org0/data/msp`即为channel msp，其中`keystore`和`signcerts`目录为空。

2. **local msp** - 保存在本地，提供链各种角色（client，orderer，peer）的身份信息。此msp需要身份文件用来签名。上图中`org0/data/orderer/msp`即为local msp表示**orderer**身份，`org0/data/admin/msp`也为local msp表示组织**admin**身份。它们下面的目录结构和组织msp下的目录结构类似，只是其中`keystore`和`signcerts`目录存有身份文件。

如果开启了**tls**安全传输则需要使用对应`tls`目录下的身份文件。

#### org1提供2个peer节点，目录结构如下

```
org1
├── config
│   ├── ica-fabric-ca-server-config.yaml      //中间CA服务配置文件
│   └── rca-fabric-ca-server-config.yaml      //根CA服务配置文件
├── data
│   ├── admin
│   │   ├── msp                               //组织admin msp目录
│   │   └── tls                               //组织admin tls目录
│   ├── msp                                   //组织msp目录
│   │   ├── admincerts
│   │   ├── cacerts
│   │   ├── intermediatecerts
│   │   ├── keystore 
│   │   ├── signcerts
│   │   ├── tlscacerts
│   │   ├── tlsintermediatecerts
│   │   └── user
│   ├── org1-ca-cert.pem                      //组织根CA证书
│   ├── org1-ca-chain.pem                     //组织中间CA证书
│   ├── peer
│   │   ├── peer1
│   │   │   ├── msp                           //组织peer1 msp目录
│   │   │   └── tls                           //组织peer1 tls目录
│   │   └── peer2
│   │       ├── msp                           //组织peer2 msp目录
│   │       └── tls                           //组织peer2 tls目录
│   └── user
│       ├── msp
│       └── tls
├── docker-compose.yml
├── start.sh
└── stop.sh
```

其目录结构和**org0**基本相同，唯一区别是两个**peer**身份目录代替了一个**orderer**身份目录。

根CA服务配置参数，中间CA服务配置参数以及调用它们生成相应的身份目录，请阅读源代码，这里不再叙述。

### 此过程的核心要点以及关键注意事项。

#### I. fabric ca server及fabric ca client调用流程为
1. 指定**fabric ca server** 启动用户名和密码，启动`fabric-ca-server`

2. 指定`FABRIC_CA_CLIENT_HOME`目录（此目录表明当前执行`fabric-ca-client`命令的身份），调用`fabric-ca-client enroll`方法生成身份证书。

3. 调用`fabric-ca-client register`方法，生成新用户

4. 指定`FABRIC_CA_CLIENT_HOME`指向新用户目录，调用`fabric-ca-client enroll`方法生成新用户身份证书。可以生成两种证书，**标准msp身份**和**tls连接用身份**。默认为生成**msp身份**，`--enrollment.profile tls`表示生成**tls身份**。

#### II. 身份证书可以使用[在线工具](https://report-uri.com/home/pem_decoder)查看内容。

下面给出几种证书内容示例，注意$\color{red}{\textbf{红色}}$标记部分

##### 中间CA证书
![](https://upload-images.jianshu.io/upload_images/14121537-c1d6d59c6bbc3936.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 用户证书
![](https://upload-images.jianshu.io/upload_images/14121537-dcf76d97cd7a066b.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### peer tls证书
![](https://upload-images.jianshu.io/upload_images/14121537-7085782acf3d34fa.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Subject Alternative Names**由`--csr.hosts $PEER_HOST`指定。它的值，**Common Name**值以及证书相关命名建议采用域名命名规范。从前到后，范围越来越大，并且不要包含`_`这种特殊字符。

#### III. 修改证书中的默认names

**Hyperledger Fabric证书中的默认names**为 `/C=US/ST=North Carolina/O=Hyperledger/OU=Fabric`，可以自定义，Demo中改为`/C=CN/ST=Si Chuan/L=Cheng Du/O=My Fabric Demo/`。

**CA证书中的names**由CA服务配置文件中`csr`部分指定，**其它生成证书的names**由`--csr.names`指定。

本篇完，请继续阅读[Hyperledger Fabric定制化联盟链搭建教程](https://www.jianshu.com/p/4fd103dee864)系列第二篇 - [MyFabric Demo链节点启动及合约部署调用]()