---

title: Ethereum私有链的搭建和部分源码编译问题的汇总

date: 2018-04-13

tags: blockchain
---

![](http://pccmxww5q.bkt.clouddn.com/ethereum.jpeg?imageView2/0/w/560/h/380/q/100)

<!-- more -->

### 准备

1.安装curl,git

2.安装go配置环境变量

```javascript
# go config
export GOPATH=$HOME/go
export GOBIN=$HOME/go/bin
export GOROOT=/usr/local/go
export PATH=/usr/local/go/bin:$GOPATH/bin:$PATH
```

3.安装nodejs+npm

### 1.Ethereum环境安装

```javascript
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo add-apt-repository -y ppa:ethereum/ethereum-dev
sudo apt-get update
sudo apt-get install ethereum
```

此时,安装的以太坊客户端版本号是最新稳定版.
对于1.5.9-stable版本的geth,我们需要通过编译源码获得.

```javascript
git clone https://github.com/ethereum/go-ethereum
cd go-ethereum
git checkout v1.5.9 #切换版本
make geth #编译
```

### 2.solc源码编译安装

```javascript
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```

此时,安装的solc也是最新版本.
旧版本的solc需要通过编译源码的方式获得.

```javascript
git clone https://github.com/ethereum/solidity.git
cd solidity
git checkout v0.4.10 #切换版本
sudo ./scripts/install_deps.sh #安装依赖
sudo ./scripts/build.sh #源码编译
```

> Ps:在Ubuntu16.04本地环境下,源码编译成功.但在Ubuntu16.04服务器环境下,编译失败,寻因未果.
> 报错信息如下:

```javascript
-- Configuring incomplete, errors occurred!
See also "/home/ubuntu/solidity/build/CMakeFiles/CMakeOutput.log".
See also "/home/ubuntu/solidity/build/CMakeFiles/CMakeError.log".
Failed to build
```

### 3.启动以太坊客户端

1.创建账户

```javascript
geth account new
geth account new
geth account new #连续创建三个账户,密码123456
```

2.生成创世区块

- 编辑创世区块配置文件

```javascript
{
    # ~/test-genesis.json
    "nonce": "0x0000000000000042",
    "difficulty": "0x1",
    "alloc": {
        "246d779602e185a53240a6cc5b01f74a17bd5deb": { #实际账户地址,需自定义
            "balance": "20000009800000000000000000000" #每个账户余额初始值
        },
        "c0a07133e5dc82ab923bbe9506f3ea4479d2b797": {
            "balance": "20000009800000000000000000000"
        },
        "b62b9d792e9825072f3d416569cc07e43a20c84f": {
            "balance": "20000009800000000000000000000"
        }
    },
    "config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000", "coinbase": "0x0000000000000000000000000000000000000000",
    "timestamp": "0x00",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "extraData": "0x11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa",
    "gasLimit": "0x2fefd8"
}
```

- 生成创世区块

```javascript
geth --datadir "~/.ethereum" init blockchain/test-genesis.json
```

3.配置以太坊客户端启动脚本

- 保存账号密码到password文件,用于解锁账户

```javascript
# 配置密码,每一个账户对应一行密码
# ~/.ethereum/password
123456
123456
123456
```

> 在以太坊中,出于安全机制,以太坊每隔一段时间就会锁定账户
> 所以我们可以有两种方式解锁以太坊账户
> 1.在启动脚本中读取密码文件解锁
> 2.使用以下js交互命令

```javascript
web3.eth.personal.unlockAccount(fromAddress, passWord, 10000) #解锁持续时间10000ms
```

- 编写启动脚本

```javascript
# ~/.ethereum/private_blockchain.sh
geth --rpc --rpcaddr="0.0.0.0" --rpccorsdomain="*" --unlock '0,1,2' #解锁三个账户
 --password ~/.ethereum/password  #保存密码明文的文件
 --nodiscover --maxpeers '5' --networkid '1234574' --datadir '~/.ethereum' console
```

- 启动以太坊客户端,进入JS交互式命令行

```javascript
bash private_blockchain.sh
```

### 4.智能合约的编译与部署

> 部署合约前的准备工作:
> 通过[Remix](https://remix.ethereum.org/)或[趣链开发者平台](http://www.hyperchain.cn/)编译智能合约获得ABI和BIN.
> account的balance大于0且已解锁.

- 定义bank_abi

```js
bank_abi=[{constant:false,inputs:[{name:'a',type:'uint256'}],name:'multiply',outputs:[{name:'d',type:'uint256'}],type:'function'}] #此处为实例,实际abi需自定义
```

- 创建合约

```js
bankContract = web3.eth.contract(bank_abi)
```

- 解锁账户

```js
personal.unlockAccount(eth.coinbase, '123456', 10000) #eth.coinbase是指定矿工的地址,默认是eth.accounts[0]
```

- 部署合约

```js
bank=bankContract.new({from:eth.coinbase,data:"0x60606040523415600b57fe5b5b60788061001a6000396000f300606060405263ffffffff60e060020a600035041663c6888fa181146020575bfe5b3415602757fe5b60306004356042565b60408051918252519081900360200190f35b600781025b9190505600a165627a7a7230582007a6259ba3d57941abda2e261e9a67958a3eda78b779d9dd8d42518791fddd590029"}) #此处的二进制是智能合约编译后生成的BIN,需自定义
```

- 挖矿上链
> 部署合约的过程实际也是由创建合约的账户发送的一笔交易（即eth.coinbase账户）。需要挖矿进行确认。

```js
miner.start(1) #启动一个线程(貌似)
miner.stop() #需手动停止挖矿
```

- 确认上链

```js
eth.getBlocks[1] #区块编号一般是已有区块的基础上加1
```

此时,如果usedGas参数大于0,表示挖矿成功.

### 5.webapp与以太坊平台的交互方式

> 未完待续

### 常用命令汇总

geth命令:

```javascript
geth --datadir "~/.ethereum" init ./test-genesis.json #以test-genesis.json脚本初始化创世区块,设置datadir
geth account new #新建一个账户,需要输入两次密码
geth account list #list所有的账户地址
```

js交互命令:

```javascript
eth #列出所有的账户地址,区块数以及eth命令下的函数名称
eth.coinbase #矿工地址,一般为第一个账户地址,即eth.accounts[0]
eth.accounts[0] #第一个账户地址
eth.getBlocks(0) #获取第一个区块信息
eth.getBalance(eth.accounts[0]) #获取第一个账户的余额
eth.getTransactionRecepit(xxx) #获取部署后的合约地址,参数为合约地址所在的区块中的Transaction字段
admin.setSloc('/usr/local/bin/solc') #指定solc编译器的地址
txpool.status #查看交易池信息,比如等待队列中交易的数量
miner.start(1) #开启一个线程进行挖矿操作
miner.stop() #需要手动停止挖矿
```

> 参考资料
1. [ubuntu系统搭建以太坊私有链](http://www.cnblogs.com/liangyue/p/6824858.html)
2. [以太坊合约简单部署和使用](https://www.cnblogs.com/lxcmyf/p/8489273.html#version=soljson-latest.js)