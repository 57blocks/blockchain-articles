本文为[Hyperledger Fabric定制化联盟链搭建教程](https://github.com/57blocks/blockchain-articles/blob/master/docs/Hyperledger%20Fabric%E5%BC%80%E5%8F%91/Hyperledger%20Fabric%E5%AE%9A%E5%88%B6%E5%8C%96%E8%81%94%E7%9B%9F%E9%93%BE%E6%90%AD%E5%BB%BA%E6%95%99%E7%A8%8B.md)系列第三篇，介绍如何使用Hyperledger Fabric Java SDK访问MyFabric Demo链。

本篇代码可以从Github上下载 - [myfabric-demo-java-client
](https://github.com/fftt2017/myfabric-demo-java-client)

### certificate目录

`certificate`目录包含项目运行需要的身份证书。

- **CA证书文件**。`org0-ca-chain.pem`和`org1-ca-chain.pem`是组织CA证书，访问org0或org1的节点需要，由于组织节点开启了tls验证。

- **org1组织用户身份文件**。`user_cert.pem`和`user_sk`是用户的身份证书和身份私钥，提供访问用户msp身份。`user_client.crt`和`user_client.key`是客户端认证证书和私钥，访问节点需要，由于组织节点开启了客户端验证。`admin_`开头的4个文件和`user_`文件类似，代表admin身份。

`certificate`目录中的身份证书需要使用MyFabric Demo链生成的证书。项目默认文件和[myfabric-demo-chain](https://github.com/fftt2017/myfabric-demo-chain)中的身份文件保持一致，可以直接使用。如果运行了`start-ca.sh`重新生成组织身份证书，需要复制相应文件到`certificate`目录中，可参考`certificate.bat`。

### 代码介绍

`SampleEnrollment`类实现`Enrollment`接口，**管理用户身份证书和私钥**。

`SampleUser`类实现`User`接口，**管理用户名，MSPID，Enrollment等信息**。

`Demo`类进行MyFabric Demo链的访问，主要流程如下。

1. **修改`private static final String HOST = "192.168.99.101"`为您启动MyFabric Demo链的机器IP**，如果在本机启动可为localhost。

2. **`getDemoChannel`方法生成client对象和channel对象**，`client.setUserContext(user)`指定一个SampleUser实例为客户端访问用户。channel对象添加一个orderer节点和一个peer节点。

3. **调用mycc合约的query方法**，`query(client, channel, CHAIN_CODE, "query", new String[] {"a"})`，显示a的当前值。

4. **调用mycc合约的invoke方法**，`invoke(client, channel, CHAIN_CODE, "invoke", new String[] {"a", "b", "10"})`，修改a和b的值。

5. **再次调用mycc合约的query方法**，`query(client, channel, CHAIN_CODE, "query", new String[] {"a"})`，显示a被修改后的值。

6. **遍历链上的每个块**，`blockWalker(client, channel)`，显示块信息和transaction信息。

详细信息，请阅读源代码。

本篇完，完成教程请阅读[Hyperledger Fabric定制化联盟链搭建教程](https://github.com/57blocks/blockchain-articles/blob/master/docs/Hyperledger%20Fabric%E5%BC%80%E5%8F%91/Hyperledger%20Fabric%E5%AE%9A%E5%88%B6%E5%8C%96%E8%81%94%E7%9B%9F%E9%93%BE%E6%90%AD%E5%BB%BA%E6%95%99%E7%A8%8B.md)系列。










