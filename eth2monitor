## Development
1. 合并master分支的geth所有代码
2. 合并代码时，保留“shawn”关键字注释的代码段，这是私有定制代码
3. 下载关键包：
   > go get github.com/denisenkom/go-mssqldb
4. 如下命令编译后会在根目录产生geth.exe的文件：
   > go build .\cmd\geth\
5. 构建目录：
   > E:\blockchain\ethrun\mainnet\geth.exe
   > E:\blockchain\ethrun\ethereum\jwt.hex
   > E:\blockchain\ethrun\ethereum\execution\prysm\prysm.bat
   > E:\blockchain\ethrun\ethereum\consensus，暂时空目录
6. 创建数据库
   > 创建CoinDB到```E:\blockchain\DataBase```目录下，增量256MB和64MB
   > 手动执行```D:\projects\src\github.com\ethereum\go-ethereum\monitor\sql_test.go```的函数```TestCreateTable```创建所有表
7. 运行geth
   > E:
   > cd E:\blockchain\ethrun\mainnet
   
   当前使用：
   > geth --datadir=E:\\blockchain\\ethrun\\mainnet\\data --syncmode=snap --verbosity=2 --maxpeers=30 --authrpc.jwtsecret=E:\\blockchain\\ethrun\\ethereum\\jwt.hex

   历史使用1：
   > geth --datadir=E:\\blockchain\\ethrun\\mainnet\\data --syncmode=full --gcmode=archive --verbosity=2 --authrpc.jwtsecret=E:\\blockchain\\ethrun\\ethereum\\jwt.hex

   历史使用2：
   > geth --datadir=E:\\blockchain\\ethrun\\mainnet\\data --syncmode=snap --verbosity=2 --cache=6000 --http --http.api="eth,net,engine,admin,miner" --authrpc.jwtsecret=E:\\blockchain\\ethrun\\ethereum\\jwt.hex

8. geth attach ipc
   > E:
   > cd E:\blockchain\ethrun\mainnet
   > geth.exe attach ipc:\\.\pipe\geth.ipc
9.  运行prysm，[参考](https://www.offchainlabs.com/prysm/docs/install/install-with-script):
   > E:
   
   > cd E:\blockchain\ethrun\ethereum\execution\prysm
   
   如果第一次运行可能报错：```an error has occured, set PRYSM_AUTORESTART=1 to automatically restart```运行如下可以出现错误自动重启。
   > set PRYSM_AUTORESTART=1

   当前使用1：
   > beacon-chain --execution-endpoint=http://localhost:8551 --datadir=E:\\blockchain\\ethrun\\ethereum\\Eth2 --jwt-secret=E:\\blockchain\\ethrun\\ethereum\\jwt.hex --suggested-fee-recipient=0xE6852A1107E9e153C9C74FB1b17c8dd835A08430 --checkpoint-sync-url=https://mainnet-checkpoint-sync.attestant.io/ --genesis-beacon-api-url=https://mainnet-checkpoint-sync.attestant.io/


   当前使用2：
   > prysm.bat beacon-chain --execution-endpoint=http://localhost:8551 --datadir=E:\\blockchain\\ethrun\\ethereum\\Eth2 --jwt-secret=E:\\blockchain\\ethrun\\ethereum\\jwt.hex --suggested-fee-recipient=0xE6852A1107E9e153C9C74FB1b17c8dd835A08430 --checkpoint-sync-url=https://mainnet-checkpoint-sync.attestant.io/ --genesis-beacon-api-url=https://mainnet-checkpoint-sync.attestant.io/

   历史使用1（prysm官方提供，我试过从零开始一直卡住）：
   > prysm.bat beacon-chain --execution-endpoint=http://localhost:8551 --datadir=E:\\blockchain\\ethrun\\ethereum\\Eth2 --jwt-secret=E:\\blockchain\\ethrun\\ethereum\\jwt.hex --suggested-fee-recipient=0xE6852A1107E9e153C9C74FB1b17c8dd835A08430  --checkpoint-sync-url=https://beaconstate.info --genesis-beacon-api-url=https://beaconstate.info
   
   历史使用2：
   > prysm.bat beacon-chain --execution-endpoint=http://localhost:8551 --datadir=E:\\blockchain\\ethrun\\ethereum\\Eth2 --jwt-secret=E:\\blockchain\\ethrun\\ethereum\\jwt.hex --suggested-fee-recipient=0xE6852A1107E9e153C9C74FB1b17c8dd835A08430 --checkpoint-sync-url=https://mainnet-checkpoint-sync.stakely.io --genesis-beacon-api-url=https://mainnet-checkpoint-sync.stakely.io

   总共只有--checkpoint-sync-url和--genesis-beacon-api-url参数不同，可以[参考](https://www.offchainlabs.com/prysm/docs/install/install-with-script)。

10. 防止Win11进入自动休眠
    “编辑电源选项”设置了GUI设置了两个“从不”后还是会进入睡眠，用一下命令：
    > powercfg -change -standby-timeout-ac 0
    
    > powercfg -change -standby-timeout-dc 0
    
    > powercfg -change -hibernate-timeout-ac 0
    
    > powercfg -change -hibernate-timeout-dc 0

11. 关于压缩数据(研究阶段)：
   > geth --datadir=E:\\ethrun\\mainnet\\data snapshot prune-state
   
   多次开关geth，消除错误：Failed to load snapshot                  err="head doesn't match snapshot: have... want..."


---

## TODO:
1. 实现所有交易对落地
D:\Projects\src\github.com\lxex\DappProjects\research\uniswapv3\UniswapV3Factory.sol
但是目前看起来都是调用v2的交易对兑换，所以需要观察v2v3的流动性，并分别区分入库

目前看：multicall(bytes[] data)/0c49ccbe/decreaseLiquidity((uint256,uint128,uint256,uint256,uint256))得到的流动性数量并不准，经常是参数为0，而实际并不是0

2. 监控设计相关交易对交易

3. 形成套利方案、编写套利模型

4. 熟悉最新版本以太坊交易，发送规则

5. 支持ENS用户名识别
