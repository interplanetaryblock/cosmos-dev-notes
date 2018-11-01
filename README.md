# Multicoin

## APP基础开发框架

在 cosmos-sdk 定义好的框架下面开发 APP，需要做的基本是下面几步：

1）首先是定义 APP 特有的 message 以及对应的 handler；

2）然后 New 一个 ABCI APP，并为各个 module 创建对应的 KVStore；

3）还需要将 message 和 handler 通过 Router 关联起来，为后续 message 处理铺路；

4）运行APP，完成初始化，加载存储数据，监听各种 message，timer，event，signal；

5）handler 处理 message 完成状态转换，并将结果存入 KVStore；

6）message 需要包装在 transaction 里面，transaction 需要有对应的 Decoder。

    以上这些 basecoin 基本完成，multicoin 就是在 basecoin 的基础上继续开发的。

## 编解码

问题：transaction 里面可以包含多种 message 类型，需要先 decode 为 interface，然后再根据 message 类型来分开处理。而在 golang 中没有内置的将一个 bytes 数组 decode 为一个 interface 类型的方法。

解法：Tendermint 库中引入了 amino 编解码，就是为了解决上面的问题。需要先将具体的 struct register 为一个全局唯一的名称，然后在编码的时候携带该名称，这样解码的时候就知道类型。

    在 APP 中引入新的模块时，需要同时调用对应的 RegisterWire 函数。multicoin 引入 stake 和 gov 是就调用了：stake.RegisterWire(cdc) 和 gov.RegisterWire(cdc)。

anteHandler 是全局的函数，在handler 之前执行，主要是为了验证交易和Fee。
3、使用标准的 x/auth 实现 Account 以及 签名验证（首次签名的 Account 要带上pubKey），x/bank 实现 代币的转移；使用最小权限原则，用 Mapper 封装 KVStore，用 Keeper 封装 Mapper。
4、InitChainer 在 APP 首次启动时会被 Tendermint 调用一次，用来初始化 coinbase Account。

## init 阶段分析：
1、传入 moniker；生成 nodeKey；创建 validator。
2、调用自定义的 AppInit.AppGenTx() 来完成：生成 privKey 和 secret phrase；初始化 Genesis transaction。
3、调用自定义的 AppInit.AppGenState() 来完成：初始化 stakeData；创建 Genesis Account 并配置一定的 Token 和 coin；New validator，添加 Token，为stakeData pool 添加 Token。
4、生成 genesis.json, config.toml, 写入 配置文件。


