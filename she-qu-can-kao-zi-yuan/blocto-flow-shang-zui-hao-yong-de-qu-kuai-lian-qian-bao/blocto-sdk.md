# Blocto SDK

## 概述



![&#x901A;&#x8FC7;Blocto SDK &#x5B8C;&#x6210;&#x7528;&#x6237;&#x4E0E;&#x533A;&#x5757;&#x94FE;&#x7684;&#x53CB;&#x597D;&#x4EA4;&#x4E92;](../../.gitbook/assets/screen-shot-2020-08-22-at-6.48.39-pm.png)

### 

### 优势

> 最好的区块链体验是用户甚至没有意识到它是区块链体验。

#### 无需下载

与独立钱包相比，不需要下载。他们不用因为要使用你的DApp而主动去下载额外的应用程序或浏览器



#### 流畅的登录

用户不必经历痛苦的钱包创建过程，在这个过程中，他们会被种子短语、私钥、地址等新概念吓到。他们可以在以后学习，因为他们在钱包里积累了更多的资产。



#### 免费增值模式

与其他钱包sdk相比，免费增值模式成为可能，你可以为用户支付交易费用，以提供免费增值体验。他们不需要在试用dApp之前购买加密货币来支付费用。 



#### 综合支付

使用Blocto的支付api轻松获得支付。用户可以轻松地使用信用卡或其他加密货币，如比特币、以太坊、波场、USDT等进行支付。一旦集成了Blocto SDK，您的用户可以通过Blocto应用程序轻松安全地管理他们的资产。您的dApp可以立即进入庞大的区块链生态系统。

#### 

#### 支持多链多种SDK

Blocto SDK支持流、以太坊、波场和未来会有更多的链。Blocto还帮助用户在不同的链上转换资产。可在不同语言和不同平台上使用。大多数钱包sdk只支持JavaScript。Blocto SDK有JavaScript、Swift \(iOS\)和Kotlin \(Android\)版本。





## Blocto SDK 快速示例

使用 Blocto 钱包服务快速开发一个简单的dApp 实例。

{% hint style="info" %}
你需要先安装 [Node.js](https://nodejs.org/zh-tw/download/package-manager/) 和 [Yar](https://classic.yarnpkg.com/en/docs/install/#windows-stable) 
{% endhint %}

### 第一步，创建应用程序和安装依赖:

```
$ npx create-react-app hello-world
```

在您刚刚创建的`hello-world`文件夹中，安装此项目所需的依赖项。

```text
$ yarn add @onflow/fcl
$ yarn add styled-components
```

{% hint style="danger" %}
FCL正在进行大量的开发，并且版本并不总是向后兼容的。我建议你现在使用@onflow/fcl@^0.0.52。
{% endhint %}

你可以启动项目，并访问地址在`http://localhost:3000`

```text
$ yarn start
```

### 第二步， 从Flow中读取数据

1. 创建`src/GetLatestBlock.js`
2. 增加组件到 `src/App.js`
3. 增加FCL的配置到 `src/index.js` 让FCL知道从哪个节点中读取数据

{% tabs %}
{% tab title="src/GetLatestBlock.js" %}
```jsx
import React, {useState} from "react"
import * as fcl from "@onflow/fcl"
import styled from 'styled-components'

const Card = styled.div`
  margin: 10px 5px;
  padding: 10px;
  border: 1px solid #c0c0c0;
  border-radius: 5px;
`

const Code = styled.pre`
  background: #f0f0f0;
  border-radius: 5px;
  max-height: 150px;
  overflow-y: auto;
  padding: 5px;
`

const GetLatestBlock = () => {
  const [data, setData] = useState(null)

  const runGetLatestBlock = async (event) => {
    event.preventDefault()

    const response = await fcl.send([
      fcl.getLatestBlock(),
    ])
    
    setData(await fcl.decode(response))
  }

  return (
    <Card>
      <button onClick={runGetLatestBlock}>
        最新区块高度
      </button>
      
      {data && <Code>{JSON.stringify(data, null, 2)}</Code>}
    </Card>
  )
}

export default GetLatestBlock

```
{% endtab %}

{% tab title="src/App.js" %}
```jsx
import React from 'react';
import styled from 'styled-components'

import GetLatestBlock from './GetLatestBlock'

const Wrapper = styled.div`
  font-size: 13px;
  font-family: Arial, Helvetica, sans-serif;
`;

function App() {
  return (
    <Wrapper>
      <GetLatestBlock />
    </Wrapper>
  );
}

export default App

```
{% endtab %}

{% tab title="src/index.js" %}
```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import * as fcl from "@onflow/fcl"
import './index.css'
import App from './App'
import * as serviceWorker from './serviceWorker'

fcl.config()
  .put("accessNode.api", "https://access-testnet.onflow.org") // 连接Flow测试网

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
)

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister()

```
{% endtab %}
{% endtabs %}

当用户点击 `最新区块高度` 按钮时，会向 Flow 的节点发出请求，获取 区块的最新数据。

### 第三步， 连接 Blocto 钱包

现在，让我们添加登录功能给你的dApp

1. 创建 `src/Authenticate.js`
2. 添加组件 `src/App.js`
3. 添加FCL的配置到 `src/index.js` 让 FCL 知道我们选择哪个钱包去使用。 

{% tabs %}
{% tab title="src/Authenticate.js" %}
```jsx
import React, {useState, useEffect} from "react"
import styled from "styled-components"
import * as fcl from "@onflow/fcl"

const Card = styled.div`
  margin: 10px 5px;
  padding: 10px;
  border: 1px solid #c0c0c0;
  border-radius: 5px;
`

const SignInOutButton = ({ user: { loggedIn } }) => {
  const signInOrOut = async (event) => {
    event.preventDefault()

    if (loggedIn) {
      fcl.unauthenticate()
    } else {
      fcl.authenticate()
    }
  }

  return (
    <button onClick={signInOrOut}>
      {loggedIn ? '退出登录' : '登录/注册'}
    </button>
  )
}

const CurrentUser = () => {
  const [user, setUser] = useState({})

  useEffect(() =>
    fcl
      .currentUser()
      .subscribe(user => setUser({...user}))
  , [])

  return (
    <Card>
      <SignInOutButton user={user} />
    </Card>
  )
}

export default CurrentUser

```
{% endtab %}

{% tab title="src/App.js" %}
```jsx
import React from 'react';
import styled from 'styled-components'

import GetLatestBlock from './GetLatestBlock'
import Authenticate from './Authenticate'

const Wrapper = styled.div`
  font-size: 13px;
  font-family: Arial, Helvetica, sans-serif;
`;

function App() {
  return (
    <Wrapper>
      <GetLatestBlock />
      <Authenticate />
    </Wrapper>
  );
}

export default App;

```
{% endtab %}

{% tab title="src/index.js" %}
```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import * as fcl from "@onflow/fcl"
import './index.css'
import App from './App'
import * as serviceWorker from './serviceWorker'

fcl.config()
  .put("accessNode.api", "https://access-testnet.onflow.org") // connect to Flow testnet
  .put("challenge.handshake", "https://flow-wallet-testnet.blocto.app/authn") // use Blocto testnet wallet

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
)

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister()

```
{% endtab %}
{% endtabs %}

当用户点击登录按钮，FCL调用出Blocto钱包，并且用户进行注册一个新的Blocto账户或者登录到一个已经存在的账户中。一旦完成了登陆后，用户选择使用Blocto账户在aApp上，然后dApp可以获得账户的信息并展示在 `<UserProfile>`中。

### 第四步，发送一个简单的交易

最后，我们可以使用连接了Blocto钱包的账户进行Flow交易的发起。

1. 创建`src/SendTransaction.js`
2. 添加组件到 `src/App.js`

{% tabs %}
{% tab title="src/SendTransaction.js" %}
```jsx
import React, {useState} from "react"
import * as fcl from "@onflow/fcl"
import styled from 'styled-components'

const Card = styled.div`
  margin: 10px 5px;
  padding: 10px;
  border: 1px solid #c0c0c0;
  border-radius: 5px;
`

const Header = styled.div`
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 5px;
`

const Code = styled.pre`
  background: #f0f0f0;
  border-radius: 5px;
  max-height: 300px;
  overflow-y: auto;
  padding: 5px;
`

const simpleTransaction = `\
transaction {
  execute {
    log("Hello World!!")
  }
}
`

const SendTransaction = () => {
  const [status, setStatus] = useState("Not started")
  const [transaction, setTransaction] = useState(null)

  const sendTransaction = async (event) => {
    event.preventDefault()
    
    setStatus("Resolving...")

    const blockResponse = await fcl.send([
      fcl.getLatestBlock(),
    ])

    const block = await fcl.decode(blockResponse)
    
    try {
      const tx = await fcl.send([
        fcl.transaction(simpleTransaction),
        fcl.proposer(fcl.currentUser().authorization),
        fcl.payer(fcl.currentUser().authorization),
        fcl.ref(block.id),
      ])

      const { transactionId } = tx

      setStatus(`Transaction (${transactionId}) sent, waiting for confirmation`)

      const unsub = fcl
        .tx(transactionId)
        .subscribe(transaction => {
          setTransaction(transaction)

          if (fcl.tx.isSealed(transaction)) {
            setStatus(`Transaction (${transactionId}) is Sealed`)
            unsub()
          }
        })
    } catch (error) {
      console.error(error)
      setStatus("Transaction failed")
    }
  }

  return (
    <Card>
      <Header>发送交易</Header>

      <Code>{simpleTransaction}</Code>

      <button onClick={sendTransaction}>
        发送
      </button>

      <Code>状态: {status}</Code>

      {transaction && <Code>{JSON.stringify(transaction, null, 2)}</Code>}
    </Card>
  )
}
export default SendTransaction

```
{% endtab %}

{% tab title="src/App.js" %}
```jsx
import React from 'react';
import styled from 'styled-components'

import GetLatestBlock from './GetLatestBlock'
import Authenticate from './Authenticate'
import SendTransaction from './SendTransaction'

const Wrapper = styled.div`
  font-size: 13px;
  font-family: Arial, Helvetica, sans-serif;
`;

function App() {
  return (
    <Wrapper>
      <GetLatestBlock />
      <Authenticate />
      <SendTransaction />
    </Wrapper>
  );
}

export default App;

```
{% endtab %}
{% endtabs %}

当用户点击 `发送` 按钮时，FCL 调用Blocto 弹出请求给用户确认是否发送交易或者拒绝。如果用户授权了交易，Blocto 钱包用key对交易进行签名，交易和签名结果发送至Flow 网络中。

{% hint style="info" %}
非常好! 你通过Blocto 钱包成功向Flow 测试网发送了一个交易!
{% endhint %}

你可以在这里找到完整的代码示例:

 [https://github.com/portto/flow-hello-world](https://github.com/portto/flow-hello-world)

## 登录和注册

通过Flow Client Library \(FCL\)连接Blocto wallet

### 第一步，配置 FCL 

```javascript
import * as fcl from "@onflow/fcl"

fcl.config()
  // connect to Flow testnet
  .put("accessNode.api", "https://access-testnet.onflow.org")
  // use Blocto testnet wallet
  .put("challenge.handshake", "https://flow-wallet-testnet.blocto.app/authn")
```

### 第二步， 授权验证

```javascript
import * as fcl from "@onflow/fcl"

fcl
  .currentUser()
  .subscribe(console.log) // fires everytime account connection status updates
  
// authenticate
fcl.authenticate()
```

### 第三步， 取消授权

```javascript
import * as fcl from "@onflow/fcl"

fcl
  .currentUser()
  .subscribe(console.log) // fires everytime account connection status updates
  
// unauthenticate and clear account info in FCL
fcl.unauthenticate()
```

## 签名交易

通过Flow 客户端库\(FCL\)与Blocto钱包签署消息

### 第一步，发送交易

```javascript
import * as fcl from "@onflow/fcl"

const SIMPLE_TRANSACTION = `\
transaction {
  execute {
    log("Hello World!!")
  }
}
`

// Get latest block info
const blockResponse = await fcl.send([
  fcl.getLatestBlock(),
])

// Decode block info
const block = await fcl.decode(blockResponse)

// Send transaction
const tx = await fcl.send([
  fcl.transaction(SIMPLE_TRANSACTION),
  fcl.proposer(fcl.currentUser().authorization),
  fcl.payer(fcl.currentUser().authorization),
  fcl.ref(block.id),
])

```

### 第二步， 监听交易结果

```javascript
import * as fcl from "@onflow/fcl"

// Monitor transaction result
fcl
  .tx(tx)
  .subscribe(console.log) // fires everytime tx status updates
```

## Flow 网络

## 项目 Demo 

更多的集成示例可以在这里找到:

 [https://github.com/portto/fcl-demo](https://github.com/portto/fcl-demo)  


在线 demo: [http://fcl-demo.portto.io/](http://fcl-demo.portto.io/)

{% hint style="success" %}
demo 项目也可以演示了 本地 **Flow emulator** 和 **dev-wallet 的使用**
{% endhint %}

## 其他例子

* **Flow FT创建**
* **NFT 创建** 
* **NFT 交易市场**
* **Flow 稳定币 FUSD**

\*\*\*\*

