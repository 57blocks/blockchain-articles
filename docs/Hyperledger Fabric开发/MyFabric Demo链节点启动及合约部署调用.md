本文为[Hyperledger Fabric定制化联盟链搭建教程](https://www.jianshu.com/p/4fd103dee864)系列第二篇，介绍MyFabric Demo链的启动和合约部署以及调用

本篇代码可以从Github上下载 - [myfabric-demo-chain](https://github.com/fftt2017/myfabric-demo-chain)

### 代码目录结构如下

```
myfabric-demo-chain
├── org0                  
├── org0-orderer     //org0 orderer 目录
├── org1                
├── org1-peer        //org1 peer目录
├── org2                  
├── org2-peer        //org2 peer目录
├── peer-cli         //peer客户端目录
├── scripts
├── start-ca.sh         
├── start-chain.sh   //链启动脚本
├── start.sh
├── stop-ca.sh         
├── stop-chain.sh    //链结束脚本
└── stop.sh
```

本节教程相关部分已用`//`标出。

`start-chain.sh`脚本依次调用`org0-orderer`，`org1-peer`，`org2-peer，peer-cli下`的`start.sh`脚本
`stop-chain.sh`脚本依次调用`peer-cli`，`org2-peer`，`org1-peer`，`org0-orderer`下的`stop.sh`脚本

#### org0-orderer启动1个orderer，目录结构如下

```
org0-orderer
├── config
│   └── configtx.yaml                    //链配置文件
├── data
│   └── genesis.block                    //orderer初始块
├── docker-compose.yml
├── fabric-ca-orderer.dockerfile
├── start.sh
└── stop.sh
```

`docker-compose`启动2个docker，orderer文件生成docker根据链配置文件生成orderer初始块，orderer节点docker

#### org1-peer启动2个peer，目录结构如下

```
org1-peer
├── docker-compose.yml
├── fabric-ca-peer.dockerfile
├── start.sh
└── stop.sh
```

`docker-compose`启动2个docker，peer1-org1节点docker，peer2-org1节点docker

#### org2-peer启动2个peer，目录结构和org1-peer类似

#### peer-cli启动peer客户端并执行合约部署调用脚本，目录结构如下

```
peer-cli
├── chaincode
│   ├── abac                    //示例合约Go源代码
│   └── github.com
├── config
│   └── configtx.yaml           //链配置文件
├── data
│   ├── channel.tx              //通道初始事务
│   ├── org1_anchors.tx         //org1设置锚节点事务
│   └── org2_anchors.tx         //org2设置锚节点事务
├── docker-compose.yml
├── start.sh
└── stop.sh
```

`docker-compose`启动2个docker

- **通道文件生成docker**，它根据链配置文件生成通道初始事务，org1设置锚节点事务，org2设置锚节点事务。

- **peer客户端docker**，它运行合约部署调用脚本，执行以下步骤

	+ 基于通道初始事务创建通道并生成通道初始块
	+ 基于通道初始块各peer节点加入通道
	+ 修改通道配置设置org1锚节点
	+ 修改通道配置设置org2锚节点
	+ 在peer节点上安装合约
	+ 在通道上初始化合约
	+ 在通道上调用合约方法

上述过程中的细节，请阅读源代码，这里不再叙述。

### 此过程的核心要点以及关键注意事项。

1. **链配置文件`configtx.yml`中`MSPDir`指向上一教程生成的组织msp目录**，**NAME**和**ID**中不要包含`_`这种特殊字符。`DemoOrdererGenesis`定义Orderer和Consortiums，`DemoChannel`基于指定的Consortium定义通道的参与组织。

2. **生成orderer genesis block时可以指定系统通道的名称**，如不提供默认委托test-channel。

3. **本Demo开启了tls和client auth**。**针对orderer**，指定`ORDERER_GENERAL_TLS_`为前缀的4个环境变量来开启tls，指定`ORDERER_GENERAL_TLS_CLIENT`为前缀的2个环境变量来开启client auth。**针对peer**，指定`CORE_PEER_TLS_`为前缀的4个环境变量来开启tls，指定`CORE_PEER_TLS_CLIENT`为前缀的4个环境来开启client auth。当peer作为client访问orderer时，需要`CORE_PEER_TLS_CLIENTKEY_FILE`和`CORE_PEER_TLS_CLIENTCERT_FILE`，当它作为server接受peer命令访问是，不需要这2个环境变量。

4. **peer上安装的合约以docker方式运行**，`CORE_VM_ENDPOINT`和`CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE`指定合约docker运行的网络，确保和peer在同一网络中。

5. **peer启动时可以通过`CORE_PEER_GOSSIP_BOOTSTRAP`指定同一组织下的其它节点**，直接加入交换节点列表。

6. **生成channel genesis block时可以指定通道名称**。

7. **peer客户端由`CORE_PEER_LOCALMSPID`决定调用者组织**，由`CORE_PEER_MSPCONFIGPATH`决定调用者身份，由`CORE_PEER_ADDRESS`决定调用节点。tls需要`CORE_PEER_TLS_ROOTCERT_FILE`。client auth需要`CORE_PEER_TLS_CLIENTKEY_FILE`和`CORE_PEER_TLS_CLIENTCERT_FILE`。

8. **本Demo采用默认权限配置**。创建通道、加入通道、更改通道、安装合约、初始化合约操作需要**admin**权限，调用合约方法普通权限即可。可以修改链配置文件自定义各种权限。

9. **合约docker镜像根据节点名、合约名、合约版本生成**，如果更新合约代码但并没有修改版本号，需要删除已有的docker镜像，重新生成才会生效。

本篇完，请继续阅读[Hyperledger Fabric定制化联盟链搭建教程](https://www.jianshu.com/p/4fd103dee864)系列第三篇 - [MyFabric Demo Java SDK访问示例]()

