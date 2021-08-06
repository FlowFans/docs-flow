# FCL Dev Wallet 的使用

### 重要提示

> FCL 开发钱包 目前正在积极的开发，它的当前状态应该被认为只适用于开发环境。
>
> 如果你遇到任何问题，请确保你是使用最新的版本，如果问题仍然存在，请通过创造一个issue让我们知道。
>
> 这个项目，虽然它确实实现了一个FCL兼容接口，目前不应作为参考建立一个生产级钱包，特别是它的处理帐户创建和私钥的方式。
>
> 这项工程只能用于帮助本地开发的用户，针对Flow的本地运行实例进行开发，区块链类似于模拟器，不应该在Flow Mainnet,  Testnet,  Canarynet或任何其他您无法完全控制的其他Flow实例中使用。



### 启动项目

```text
git clone https://github.com/onflow/fcl-dev-wallet.git
cd fcl-dev-wallet
cp .env.example .env.local

# 更新 .env.local 中的值以匹配您的本地环境

yarn install
yarn dev
```

配置文件信息：

{% tabs %}
{% tab title=".env.local " %}
```text
# FCL开发钱包需要知道如何与Flow模拟器交互
# 因为 FCL开发钱包需要通过 Javascript 使用 FCL 与 emulator 进行交互
# 所以设定 emulators grpc-http proxy， 一般不必修改
FLOW_ACCESS_NODE=http://localhost:8080

# FCL开发钱包需要一个帐户作为基础/起点。
# 此帐户将用于创建和管理其他帐户。
# 我们建议使用Flow中定义的服务帐户， 该账户被定义在 flow.json 文件中，也是你本地模拟器正在使用的账户。 
FLOW_ACCOUNT_ADDRESS=0xf8d6e0586b0a20c7
FLOW_ACCOUNT_KEY_ID=0
FLOW_ACCOUNT_PRIVATE_KEY=84f82df6790f07b281adb5bbc848bd6298a2de67f94bdfac7a400d5a1b893de5
FLOW_ACCOUNT_PUBLIC_KEY=4519e9fbf966c6589fafe60903c0da5f55c5cb50aee5d870f097b35dfb6de13c170718cd92f50811cdd9290e51c2766440b696e0423a5031ae482cca79e3c479



```
{% endtab %}
{% endtabs %}



