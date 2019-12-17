# Gbif单节点部署文档
### 单机部署gbif节点大致分为几个步骤：
 - 配置好go运行环境，推荐使用CentOS7
 - 配置gbif源码，本地编译通过
 - 执行gbif初始化操作
 - 启动gbif节点
 - 创建一个并且解锁账号，用于挖矿

### 推荐服务器
- CentOS7
- 4核
- 16G
- 50G

### 节点安装流程
#### 1. 配置go运行环境：
#### 下载go包
```shell
 wget https://dl.google.com/go/go1.13.5.linux-amd64.tar.gz
 tar -C /usr/local -xzf go1.13.5.linux-amd64.tar.gz
```

#### 2. 配置go环境变量，在~/.bashrc中添加配置
```shell
vim ~/.bashrc
//添加到最后
export GOROOT=/usr/local/go
export GOPATH=$HOME/code
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
export GO111MODULE=off

source ~/.bashrc

go version //输出 go version go1.13.5 linux/amd64，代表安装成功
```

#### 3. 配置go-bif源码,执行完成后，会在项目根目录生成gbif客执行文件
```shell
mkdir -p $HOME/code/src/github.com/bif
cd $HOME/code/src/github.com/bif
git clone http://192.168.150.43:8080/BIF/go-bif.git
cd go-bif
go build github.com/bif/go-bif/cmd/gbif
```

#### 4. 配置genesis.json文件
在项目跟目录下添加genesis.json文件，文件内容如下：
```json
{
  "config": {
    "chainId": 444,
    "constantinopleBlock": 1,
    "dpos": {
      "period": 2,
      "witnessesNum": 3,
      "witnesses": [
        {
          "address": "did:bid:6cc796b8d6e2fbebc9b3cf9e",
          "url": "enode://862d66d1255c7ac9c6d86920081069a4d7c295c838140217644ac6f4fcddd0d2edef63951148affb107223fbd1331363e275125bc0cfeb34b1daad26328f8b20@127.0.0.1:44001?discport=0"
        },
        {
          "address": "did:bid:c935bd29a90fbeea87badf3e",
          "url": "enode://0d843e75752ca5c7db12c2e8e2dc4e6eb771ddf1511809c4e708bc74abad0f14f970229a64287ec34122d3657167ea7dd5c9f1902ca6fcfc0b0a91e78d39573c@127.0.0.1:44011?discport=0"
        },
        {
          "address": "did:bid:13803fb30b7e95d57103c2dc",
          "url": "enode://7e42b87fda19dfe6b4a8e425d882c60a43667a3d731aabd19ad9aa3cdb98f0cc76c2664cb12b1a39c2c2e848afc6262a73a800cd845af10de1e90766628b2cc8@127.0.0.1:44021?discport=0"
        }
      ]
    }
  },
  "nonce": "0x0",
  "timestamp": "0x5d131141",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000f885f83f946469643a6269643a6cc796b8d6e2fbebc9b3cf9e946469643a6269643ac935bd29a90fbeea87badf3e946469643a6269643a13803fb30b7e95d57103c2dcb8410000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000c0",
  "gasLimit": "0x47b760",
  "difficulty": "0x1",
  "mixHash": "0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
  "coinbase": "did:bid:000000000000000000000000",
  "alloc": {
    "did:bid:6cc796b8d6e2fbebc9b3cf9e": {
      "balance": "0x446c3b15f9926687d2c40534fdb564000000000000"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```
#### 5. 初始化gbif
```shell
./gbif init genesis.json --datadir ./data
```

#### 6. 启动节点
```shell
./gbif --networkid 444 --datadir ./data --verbosity 4 --port 44001 --rpc --rpcaddr 0.0.0.0 --rpcport 44002 --rpcapi core,gb,admin,personal,db,net,txpool,bif --ws --wsaddr 0.0.0.0 --wsport 44003 --wsapi core,gb,admin,personal,db,net,txpool --rpccorsdomain '*' console
```
- networkid： gbif网络号，要和genesis.json里面的一致
- --verbosity： 日志等级
- --rpcport： rpc端口号
- --rpcapi： 根据需要选择开放的rpc类型
- --wsapi： 根据需要选择开放websocket api类型
- --wsport： websocket端口号
- console： 添加console后，节点启动后将自动进入控制台(gbif console)界面

##### 7. 启动后会看到如下输出：
```shell
Welcome to the Gbif JavaScript console!

instance: Gbif/v1.9.0-unstable/darwin-amd64/go1.13.5
at block: 0 (Wed, 26 Jun 2019 14:31:29 CST)
 datadir: /home/teleinfo/code/src/github.com/bif/go-bif/data/
 modules: admin:1.0 bif:1.0 core:1.0 debug:1.0 dpos:1.0 gb:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0

> 
> 
>
```
此时我们已成功启动节点，并且已进入到控制台界面

##### 8. 配置挖矿账号
在刚刚出现的控制台中输入, teleinfo 为您的账号密码，可以随意定制，还请记录好：
```shell
personal.newAccount("teleinfo")
personal.unlockAccount(core.accounts[0], "teleinfo")
core.accounts[0] //查看刚刚生成的地址
admin.nodeInfo.enode //查看enode 
miner.start()
```
此时，在data/keystore下会有一个文件名为： UTC--2019-12-16T11-18-04.703989000Z--did-bid-6cc796b8d6e2fbebc9b3cf9e 这种格式的文件，此文件即为您的keystore，文件名最后面为地址
enode格式为: "enode://4b87eed1cbb9c85abf94f784daa63ed4358726f5fefe702fea77cbce1a53813b68139f02779c493f87b3566b34cf35a5bf2963dcad8b5fee00f4be741b56f864@127.0.0.1:44031"

#### 此时，节点已经启动好，只需等待它自动加入到主网中


# Gbif本地私有链部署
本地私有链是在本地电脑上部署出一整套完成的gbif的节点，这里以四个节点组成一条链为例：
 - 三个超级节点 
 - 一个普通节点


超级节点和普通节点配置时候的区别在于genesis.json中"witnesses"和"extraData"字段中是否包含当前节点的address和enode，而启动自己的私有链的第一步就是生成自己的genesis.json. 我们先拿到一个genesis.json的模板：
```json
{
  "config": {
    "chainId": 444,
    "constantinopleBlock": 1,
    "dpos": {
      "period": 2, //出块时间,单位为秒
      "witnessesNum": 3, //见证人数量，这里是以4个几点为例，根据3m+1规则,3个即为dpos的最小数量了，所以这里设定为3
      "witnesses": [
          //TODO
      ]
    }
  },
  "nonce": "0x0",
  "timestamp": "0x5d131141",
  "extraData":  //TODO,
  "gasLimit": "0x47b760",
  "difficulty": "0x1",
  "mixHash": "0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
  "coinbase": "did:bid:000000000000000000000000",
  "alloc": {
    //TODO
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```
上面标出//TODO的地方即为我们需要自己生成的部分，可以看到有三处
### 自己生成witnesses
按照上面单节点的部署流程，我们在走到第3步之后，直接执行第6步，即可以拿到节点的address和enode
需要注意的是，启动的时候datadir和端口号需要修改一下，启动三个命令行窗口，都cd 到项目录，然后分别执行下面的命令
```shell
//命令行窗口1
./gbif --networkid 444 --datadir ./data/node0 --verbosity 4 --port 44001 --rpc --rpcaddr 0.0.0.0 --rpcport 44002 --rpcapi core,gb,admin,personal,db,net,txpool,bif --ws --wsaddr 0.0.0.0 --wsport 44003 --wsapi core,gb,admin,personal,db,net,txpool --rpccorsdomain '*' console

//命令行窗口2
./gbif --networkid 444 --datadir ./data/node1 --verbosity 4 --port 44011 --rpc --rpcaddr 0.0.0.0 --rpcport 44012 --rpcapi core,gb,admin,personal,db,net,txpool,bif --ws --wsaddr 0.0.0.0 --wsport 44013 --wsapi core,gb,admin,personal,db,net,txpool --rpccorsdomain '*' console

//命令行窗口3
./gbif --networkid 444 --datadir ./data/node2 --verbosity 4 --port 44021 --rpc --rpcaddr 0.0.0.0 --rpcport 44022 --rpcapi core,gb,admin,personal,db,net,txpool,bif --ws --wsaddr 0.0.0.0 --wsport 44023 --wsapi core,gb,admin,personal,db,net,txpool --rpccorsdomain '*' console

```
然后按照上面第6步的方法，分别拿到各自的address和enode，按照下面的格式添加到genesis.json, "witenesses"节点下：
```json
    {
        "address": "did:bid:6cc796b8d6e2fbebc9b3cf9e", //命令行窗口1拿到的address
        "url": "enode://862d66d1255c7ac9c6d86920081069a4d7c295c838140217644ac6f4fcddd0d2edef63951148affb107223fbd1331363e275125bc0cfeb34b1daad26328f8b20@127.0.0.1:44001?discport=0" //命令行窗口1拿到的enode
    },
    {
        "address": "did:bid:c935bd29a90fbeea87badf3e", //命令行窗口2拿到的address
        "url": "enode://0d843e75752ca5c7db12c2e8e2dc4e6eb771ddf1511809c4e708bc74abad0f14f970229a64287ec34122d3657167ea7dd5c9f1902ca6fcfc0b0a91e78d39573c@127.0.0.1:44011?discport=0" //命令行窗口2拿到的enode
    },
    {
        "address": "did:bid:13803fb30b7e95d57103c2dc", //命令行窗口3拿到的address
        "url": "enode://7e42b87fda19dfe6b4a8e425d882c60a43667a3d731aabd19ad9aa3cdb98f0cc76c2664cb12b1a39c2c2e848afc6262a73a800cd845af10de1e90766628b2cc8@127.0.0.1:44021?discport=0" //命令行窗口3拿到的enode
    }
```
### 编写config.toml， 生成extraData
```shell
//当前目录为项目根目录
vim config.toml

//向文件中写入(这三个地址依次的上面三个节点的地址)：
vanity = "0x00"
validators = ["did:bid:6cc796b8d6e2fbebc9b3cf9e","did:bid:c935bd29a90fbeea87badf3e","did:bid:13803fb30b7e95d57103c2dc"]
```
生成extraData：
```shell
./gbif extra encode --config ./config.toml
//生成的结果为如下格式：
//Encoded Istanbul extra-data: 0x0000000000000000000000000000000000000000000000000000000000000000f885f83f946469643a6269643a6cc796b8d6e2fbebc9b3cf9e946469643a6269643ac935bd29a90fbeea87badf3e946469643a6269643a13803fb30b7e95d57103c2dcb8410000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000c0

```
将0x00000000...00000c0 一长串放入到genesis.json的"extraData"中

### 配置alloc
alloc 即为链启动后创世产生的token，这里配置多少地址和对应的token，链产生后，这些地址上就会有相应多的token，填写格式如下：
```shell
    "did:bid:6cc796b8d6e2fbebc9b3cf9e": { //要创世时发配token的地址
      "balance": "0x446c3b15f9926687d2c40534fdb564000000000000" //设定这个地址是多少钱
    }
```

### 最终组合成的genesis.json格式如下
```json
{
  "config": {
    "chainId": 444,
    "constantinopleBlock": 1,
    "dpos": {
      "period": 2,
      "witnessesNum": 3,
      "witnesses": [
        {
          "address": "did:bid:6cc796b8d6e2fbebc9b3cf9e",
          "url": "enode://862d66d1255c7ac9c6d86920081069a4d7c295c838140217644ac6f4fcddd0d2edef63951148affb107223fbd1331363e275125bc0cfeb34b1daad26328f8b20@127.0.0.1:44001?discport=0"
        },
        {
          "address": "did:bid:c935bd29a90fbeea87badf3e",
          "url": "enode://0d843e75752ca5c7db12c2e8e2dc4e6eb771ddf1511809c4e708bc74abad0f14f970229a64287ec34122d3657167ea7dd5c9f1902ca6fcfc0b0a91e78d39573c@127.0.0.1:44011?discport=0"
        },
        {
          "address": "did:bid:13803fb30b7e95d57103c2dc",
          "url": "enode://7e42b87fda19dfe6b4a8e425d882c60a43667a3d731aabd19ad9aa3cdb98f0cc76c2664cb12b1a39c2c2e848afc6262a73a800cd845af10de1e90766628b2cc8@127.0.0.1:44021?discport=0"
        }
      ]
    }
  },
  "nonce": "0x0",
  "timestamp": "0x5d131141",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000f885f83f946469643a6269643a6cc796b8d6e2fbebc9b3cf9e946469643a6269643ac935bd29a90fbeea87badf3e946469643a6269643a13803fb30b7e95d57103c2dcb8410000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000c0",
  "gasLimit": "0x47b760",
  "difficulty": "0x1",
  "mixHash": "0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
  "coinbase": "did:bid:000000000000000000000000",
  "alloc": {
    "did:bid:6cc796b8d6e2fbebc9b3cf9e": {
      "balance": "0x446c3b15f9926687d2c40534fdb564000000000000"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

此时退出上面启动的三个超级节点(在gbif console 中输入exit即可退出)，将新配置出来的genesis.json放到项目根目录下，分别打开四个命令行窗口
```shell
//窗口一：
cd $HOME/code/src/github.com/bif/go-bif
rm -rf data/node0/gbif
./gbif init genesis.json --datadir ./data/node0
./gbif --networkid 444 --datadir ./data/node0 --verbosity 4 --port 44001 --rpc --rpcaddr 0.0.0.0 --rpcport 44002 --rpcapi core,gb,admin,personal,db,net,txpool,bif --ws --wsaddr 0.0.0.0 --wsport 44003 --wsapi core,gb,admin,personal,db,net,txpool --rpccorsdomain '*' console

//进入gbif console之后执行：
personal.unlockAccount(core.accounts[0], "teleinfo")
miner.start()

//窗口二：
cd $HOME/code/src/github.com/bif/go-bif
rm -rf data/node1/gbif
./gbif init genesis.json --datadir ./data/node1
./gbif --networkid 444 --datadir ./data/node1 --verbosity 4 --port 44011 --rpc --rpcaddr 0.0.0.0 --rpcport 44012 --rpcapi core,gb,admin,personal,db,net,txpool,bif --ws --wsaddr 0.0.0.0 --wsport 44013 --wsapi core,gb,admin,personal,db,net,txpool --rpccorsdomain '*' console

//进入gbif console之后执行：
personal.unlockAccount(core.accounts[0], "teleinfo")
miner.start()

//窗口三：
cd $HOME/code/src/github.com/bif/go-bif
rm -rf data/node2/gbif
./gbif init genesis.json --datadir ./data/node2
./gbif --networkid 444 --datadir ./data/node2 --verbosity 4 --port 44021 --rpc --rpcaddr 0.0.0.0 --rpcport 44022 --rpcapi core,gb,admin,personal,db,net,txpool,bif --ws --wsaddr 0.0.0.0 --wsport 44023 --wsapi core,gb,admin,personal,db,net,txpool --rpccorsdomain '*' console

//进入gbif console之后执行：
personal.unlockAccount(core.accounts[0], "teleinfo")
miner.start()

//窗口四：
cd $HOME/code/src/github.com/bif/go-bif
./gbif init genesis.json --datadir ./data/node3
./gbif --networkid 444 --datadir ./data/node3 --verbosity 4 --port 44031 --rpc --rpcaddr 0.0.0.0 --rpcport 44032 --rpcapi core,gb,admin,personal,db,net,txpool,bif --ws --wsaddr 0.0.0.0 --wsport 44033 --wsapi core,gb,admin,personal,db,net,txpool --rpccorsdomain '*' console

//进入gbif console之后执行：
personal.unlockAccount(core.accounts[0], "teleinfo")
miner.start()
```