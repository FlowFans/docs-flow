---
description: 如何在 Flow主网创建一个Account ？
---

# 如何在 Flow主网创建一个Account ？

一直以来在Flow主网上创建账户的途径主要是通过 Blocto 钱包，通过邮箱注册和私钥托管模式进行创建。 由于Flow创建Account必须由现有账户签名授权，那对于一些开发需求来说，可能会存在两个困难，一是如何导出Blocto上Flow账户的私钥，进行自行托管。 二是如何用自托管的账户创建一个新的Flow主网的账户。

**必备条件：**

* 需要有一个主网账户 （包含一点Flow Token）
* 如果账户来自Blocto，需导出keystore文件
* 本地已安装 Flow CLI

**Flow创建Account的步骤如下：**



### 1. 拥有包含少量FlowToken的账户，作为签名（signer）账户

该账号可以从Blocto中进行导出， 导出的文件是包含 keystore 内容的PDF文件。我们需要通过“备援密码” + keystore 进行解析，获得主网账户的“私钥”。

创建一个包含 keystore 内容的json文件`key.json`, 然后安装依赖：

```text
npm i ethereum-keystore
```

完成后，创建一个`run.js`文件，在该文件中导入`key.json`：

```text
const { recoverKeystore } = require('ethereum-keystore');
const keystoreJson = require('./key.json');

const main = async () => {
 const privateKey = await recoverKeystore(keystoreJson, '从Blocto导出时设置的备援密码');
 console.log(privateKey);
};

main();
```

最后执行，`node run.js`

关于 keystore 的详细原理可查看： [https://ethfans.org/posts/what-is-an-ethereum-keystore-file](https://ethfans.org/posts/what-is-an-ethereum-keystore-file)

关于 keysotre 转换导出 “私钥”的详细过程可查看： [https://script.money/posts/027-flow\_mainnet\_cli\_transaction/](https://script.money/posts/027-flow_mainnet_cli_transaction/)

### 2. 使用 Flow CLI 本地生成密钥对

一个带有私钥的主网账户具备之后，本地来生成一个“密钥对”，为创建新账户做准备。 全保本地安装好了[Flow CLI](https://docs.onflow.org/flow-cli/generate-keys/)

```text
flow keys generate
```

会看到类似如下内容, 包含一个`公钥`和`私钥`：

```text
🔴️ Store Private Key safely and don't share with anyone! 
Private Key      c778170793026a9a7a3815dabed68ded445bde7f40a8c66889908197412be89f 
Public Key      584245c57e5316d6606c53b1ce46dae29f5c9bd26e9e8...aaa5091b2eebcb2ac71c75cf70842878878a2d650f7
```

到这里，你完成了秘钥对的创建，妥善保护好你的`私钥`。

### 3. 初始化flow.json 文件添加配置 

在当前文件夹下执行 `flow init`后， 你会发现本地创建了一个`flow.json`文件，这里面是关于一些配置的内容,我们需填写accounts里面相关的配置，如下一般默认即可，但`privateKey`\(私钥\)，是通过上面步骤获得到的：

```text
{
 "emulators": {
     "default": {
         "port": 3569,
         "serviceAccount": "emulator-account"
     }
 },
 "contracts": {},
 "networks": {
     "emulator": "127.0.0.1:3569",
     "mainnet": "access.mainnet.nodes.onflow.org:9000",
     "testnet": "access.devnet.nodes.onflow.org:9000"
 },
 "accounts": {
     "mainnet-account": {
         "address": "0xa748021e7f7addc3",
         "key": {
             "type": "hex",
             "index": 3,
             "signatureAlgorithm": "ECDSA_secp256k1",
             "hashAlgorithm": "SHA3_256",
             "privateKey": "c0146316f1d...49f83569736aeb591b8b6"
         }
     }
 },
 "deployments": {}
}
```

### 4. 使用 Flow CLI 创建一个账户

执行创建账户的命令，这里需要加一些额外的参数，保证我们能够生成的是主网中账户：

```text
flow accounts create --key 81da6d86d95ef1a2801544e4b6cf4a3eaba1b6...1d96ffd9c64233e21231c519a7506ecccdedec129cd81fedd0504b829ea14f37b1602622b7a719a29e3 --signer mainnet-account --network mainnet
```

`key`的部分，一定要填写，上面我们本地创建的秘钥对中的`公钥`。 `signer`的部分，填写我们配置文件中的`mainnet-account`部分即可。 `network`的部分，就是主网网络的节点地址，也是在`flow.json`配置文件中设置。

然后出现如下内容，表示创建账户的交易成功:

```text
Transaction ID: 7e130f0ae269b781e77053c43b66468637ddd001e9f7c22c56f7cd51d9cc75eb

Address  0x0ae342dfada66015
Balance  0.00100000
Keys     1

Key 0   Public Key               81da6d86d95ef1a2801544e4b6cf4a3eaba1b6f52baaa1d96ffd9c64233e21231c519a7506ecccdedec129cd81fedd0504b829ea14f37b1602622b7a719a29e3
        Weight                   1000
        Signature Algorithm      ECDSA_P256
        Hash Algorithm           SHA3_256
        Revoked                  false
        Sequence Number          0
        Index                    0

Contracts Deployed: 0


Contracts (hidden, use --include contracts)
```

其中 Address 就是我们的新账户： `0x0ae342dfada66015` 如果发生报错，一定注意检查，你的`signer` 账户中是否具有多余的Flow Token,至少大于`0.002`, 否则可能无法完成签名。

### 5. Flowscan 查看账户信息

最后，我们可以在主网的区块浏览器，确认一下新账户`0x0ae342dfada66015`的状态和基本信息：

Flow 主网区块浏览器： [https://flowscan.org/](https://flowscan.org/)  


### 相关资料

keystore 的详细原理： [https://ethfans.org/posts/what-is-an-ethereum-keystore-file](https://ethfans.org/posts/what-is-an-ethereum-keystore-file)

keysotre 转换 “私钥”的过程： [https://script.money/posts/027-flow\_mainnet\_cli\_transaction/](https://script.money/posts/027-flow_mainnet_cli_transaction/)

Flow 主网区块浏览器： [https://flowscan.org/](https://flowscan.org/)

Flow CLI： [https://docs.onflow.org/flow-cli/generate-keys/](https://docs.onflow.org/flow-cli/generate-keys/)

### 



