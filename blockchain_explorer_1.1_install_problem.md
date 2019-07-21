# blockchain_explorer 1.1安装问题解决

### 问题1：nodejs脚本报对象方法undefined

```
在fabric 1.1上，安装blockchain explorer,同事安装了几天，一直没有成功。反馈是跑blockchain explorer 3.5的demo可以通，但是对接我们的项目则报错。具体报错是nodejs报错，报server.on()方法未定义，由于不太懂nodejs，于是打印查看了下，原来是explorer对象创建时失败，对象不存在，因此报方法未定义。
```

分析到此，再加上回顾了下[安装blockchain-explorer过程](http://db1d50b9.wiz03.com/share/s/3r7l2V38zA5926HuGq1vEMF22E-t3-0yiQC12C7ZIi3r5lOL)，基本断定是配置问题。

具体看下配置文件：/blockchain-explorer/app/platform/fabric/config.json

```shell
{
  "network-config": {
    "org1": {
      "name": "peerOrg1",
      "mspid": "Org1MSP",
      "peer1": {
        "requests": "grpcs://127.0.0.1:7051",
        "events": "grpcs://127.0.0.1:7053",
        "server-hostname": "peer0.org1.example.com",
        "tls_cacerts":
          "fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
      },
      "peer2": {
        "requests": "grpcs://127.0.0.1:8051",
        "events": "grpcs://127.0.0.1:8053",
        "server-hostname": "peer1.org1.example.com",
        "tls_cacerts":
          "fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt"
      },
      "admin": {
        "key":"fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore",
        "cert":"fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts"
      }
    },
    "org2": {
      "name": "peerOrg2",
      "mspid": "Org2MSP",
      "peer1": {
        "requests": "grpcs://127.0.0.1:9051",
        "events": "grpcs://127.0.0.1:9053",
        "server-hostname": "peer0.org2.example.com",
        "tls_cacerts":
          "fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
      },
      "peer2": {
        "requests": "grpcs://127.0.0.1:10051",
        "events": "grpcs://127.0.0.1:10053",
        "server-hostname": "peer1.org2.example.com",
        "tls_cacerts":
          "fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt"
      },
      "admin": {
        "key":
          "fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore",
        "cert":
          "fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts"
      }
    }
  },
  "channel": "mychannel",
  "orderers": [
    {
      "mspid": "OrdererMSP",
      "server-hostname": "orderer.example.com",
      "requests": "grpcs://127.0.0.1:7050",
      "tls_cacerts":
        "fabric-path/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt"
    }
  ],
  "keyValueStore": "/tmp/fabric-client-kvs",
  "configtxgenToolPath": "fabric-path/fabric-samples/bin",
  "SYNC_START_DATE_FORMAT": "YYYY/MM/DD",
  "syncStartDate": "2018/01/01",
  "eventWaitTime": "30000",
  "license": "Apache-2.0",
  "version": "1.1"
}
```

着重看json的key，组织的key是`org1`，这里被小伙伴写成了业务里的`**up`了，导致nodejs解析不到组织数据，最终导致explorer对象和server对象的方法undefined。

### 问题2  send deliver bad_request

这个问题是deliver部分的问题，客户端向orderer deliver数据时，服务端返回bad_request。查看了下orderer容器的日志，原来是客户端时间与orderer时间有差异，时间同步后，重启，解决问题。 