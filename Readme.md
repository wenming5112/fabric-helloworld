# *作者：jockming*
# *联系方式：1299986041*
# *博客：https://www.cnblogs.com/jockming/*
# *交流群（QQ）：537487044（Fabric技术交流群）*

------

# HelloWorld案例部署指导

------

## 目录结构1


### 目录结构
########################################

```
(仅供参考)
当前所在目录是/home目录
.
└── helloworld
    ├── bin
    │   ├── configtxgen
    │   └── cryptogen
    ├── bootstrap.sh
    ├── chaincode
    │   └── go
    │       └── helloworld
    │           ├── chaincode.go
    │           └── cmd
    │               └── main.go
    ├── channel-artifacts
    │   ├── genesis.block
    │   └── mychannel.tx
    ├── configtx.yaml
    ├── crypto-config
    │   ├── ...
    ├── crypto-config.yaml
    ├── docker-oderer.yaml
    └── docker-peer.yaml
```

------

### 准备链码
########################################

Helloworld链码实现Init和Invoke两个接口，通过stub.PutState和stub.GetState保存和获取链值对数据。

1、Init(stub shim.ChaincodeStubInterface)：

- 用于智能合约初始化及升级初始化,实现初始化时保存链值对；

2、Invoke(stub shim.ChaincodeStubInterface)：

- 是节点（peer）调用链码的入口函数，实现对账本进行保存和获取链值对；

实现两个文件,分别在(文件所在位置参看 "一、目录结构")helloworld/chaincode.go 和 helloworld/cmd/main.go，main.go是主入口函数。

------

### 准备工具
########################################

工具下载地址：[点击这里](https://github.com/hyperledger/fabric/releases/download/v1.4.1/hyperledger-fabric-linux-amd64-1.4.1.tar.gz)

下载之后解压，在其文件夹中的"bin"目录下有"cryptogen"和"configtxgen"等工具。（复制到"helloworld/bin"目录中）

------

### 拉取镜像
########################################

源码下载地址：[点击这里](https://github.com/hyperledger/fabric/archive/v1.4.1.zip)

下载之后解压，在其文件夹中的"scripts"目录中有拉取镜像的脚本"bootstrap.sh"。（复制到"helloworld"目录中）

可以直接上传到服务器，执行以下命令来拉取镜像:

1、赋权

 `$ chmod +x ./bootstrap.sh`

2、执行

 `$ ./bootstrap.sh 1.4.1 -s -b`

------


### 准备配置文件
########################################

1、用于生成peer证书和ordeer证书
- `crypto-config.yaml`

2、用于生成创世区块文件和通道配置文件
- `configtx.yaml`

以上两个文件参考当前目录下的这两个文件

------


### 准备节点启动文件
########################################

1、启动orderer节点配置
- `docker-oderer.yaml`

2、启动peer和cli节点配置
- `docker-peer.yaml`

以上两个文件参考当前目录下的这两个文件

------


### 启动前的最后准备
########################################

helloworld文件目录结构参考[目录结构](#目录结构)

1. 上传"helloworld.tar"

 将这个目录下的所有文件整理，并打包上传到服务器上。

2. 解压"helloworld.tar"
 ```
 $ cd /home
 $ tar -xvf helloworld.tar && rm -rf helloworld.tar
 $ cd helloworld
 ```

3. 生成公私钥和证书

 `$ ./bin/cryptogen generate --config=./crypto-config.yaml`

4. 生成创世区块

 `$ ./bin/configtxgen -profile OneOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block`

5. 生成通道（Channel）配置区块

 `$ ./bin/configtxgen -profile OneOrgsChannel -outputCreateChannelTx ./channel-artifacts/mychannel.tx -channelID mychannel`

------

### 启动网络
########################################

1. 启动orderer节点

 `$ docker-compose -f ./docker-orderer.yaml up -d`

2. 启动peer和cli节点

 `$ docker-compose -f ./docker-peer.yaml up -d`

3. 查看启动状况

 `$ docker ps -a`

4. 查看启动日志（"ctrl + c" 可以退出日志查看）

 `$ docker logs -f 容器ID`

------


### cli中的操作
########################################

1. 进入cli容器

 `$ docker exec -it cli bash`

2. 指定排序节点的CA证书

 `$ ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem`

3. 创建通道

 `$ peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/mychannel.tx --tls --cafile $ORDERER_CA`

4. peer加入通道

 `$ peer channel join -b mychannel.block`

5. 安装智能合约

 `$ peer chaincode install -n mycc -p github.com/hyperledger/fabric/helloworld/chaincode/go/helloworld/cmd -v 1.0`

6. 实例化智能合约(添加一个记录"helloworld")

 `$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n mycc -v 1.0 -c '{"Args":["a","helloworld"]}' -P "OR ('Org1MSP.peer')"`

7. 查询智能合约

 `$ peer chaincode query -C mychannel -n mycc -c '{"Args":["get","a"]}'`


PS: 最后就可以在cli容器中看到helloworld
