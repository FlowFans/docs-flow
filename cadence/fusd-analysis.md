# FUSD 分析

### 摘要：

1. FUSD基础源码分析
2. 重点分析FUSD合约是如何授予别人mint权限并能单方面撤回的
3. 补足FUSD Github中缺失的transaction
4. 其他想法

### 建议前置知识：

1. 了解Fungible Token 标准: [https://github.com/onflow/flow-ft](https://github.com/onflow/flow-ft)
2. 熟悉Capability: [https://docs.onflow.org/cadence/language/capability-based-access-control/](https://docs.onflow.org/cadence/language/capability-based-access-control/)

### 相关链接：

FUSD官方信息：

[https://docs.onflow.org/fusd/](https://docs.onflow.org/fusd/)

[https://docs.onflow.org/fusd/transactions/](https://docs.onflow.org/fusd/transactions/)

[https://docs.onflow.org/fusd/providers/](https://docs.onflow.org/fusd/providers/)

FUSD在测试网和主网上的合约地址：

[https://flow-view-source.com/testnet/account/0xe223d8a629e49c68/contract/FUSD](https://flow-view-source.com/testnet/account/0xe223d8a629e49c68/contract/FUSD)

[https://flow-view-source.com/mainnet/account/0x3c5959b568896393/contract/FUSD](https://flow-view-source.com/mainnet/account/0x3c5959b568896393/contract/FUSD)

FUSD Github：

[https://github.com/onflow/fusd](https://github.com/onflow/fusd)

查询FUSD余额：

[https://github.com/onflow/fusd/blob/main/transactions/scripts/get_fusd_balance.cdc](https://github.com/onflow/fusd/blob/main/transactions/scripts/get_fusd_balance.cdc)

FUSD在Flow Scan：

[https://flowscan.org/contract/A.3c5959b568896393.FUSD/transfers](https://flowscan.org/contract/A.3c5959b568896393.FUSD/transfers)

### FUSD合约概览：

FUSD合约遵循Fungible Token标准，因此任何人都可以创建 `Vault Resource`，并通过 `Vault Resource`接收或转出FUSD，并且任何人都可以查询所有人的FUSD 余额(Balance)。

此外，合约的AdminAccount可以通过“授予”其他用户 `Minter Capability`以允许其他账户mint FUSD。并且，AdminAccount可以单方面通过 `unlink capability`撤回(revoke)用户的mint FUSD的权利。

### 实现细节：

合约的前半部分是主要是实现Fungible Token标准的interface，在此不做赘述。将来我们会详细分析FT和NFT标准合约。

FUSD合约的亮点是在对于Minter权限的处理上。

首先我们可以看到在 `init()`语句中，创建了 `Administrator`的 `resource`并存储在 `adminAccount`的 `storage`中。

```swift
let admin <- create Administrator()
adminAccount.save(<-admin, to: self.AdminStoragePath)
```

`Administrator` `resource`只有一个用来创建新minter的 `function`，这个function会创建并返回一个 `Minter`  `resource`

```swift
pub resource Administrator {

    // createNewMinter
    //
    // Function that creates a Minter resource.
    // This should be stored at a unique path in storage then a capability to it wrapped
    // in a MinterProxy to be stored in a minter account's storage.
    // This is done by the minter account running:
    // transactions/FUSD/minter/setup_minter_account.cdc
    // then the admin account running:
    // transactions/flowArcaddeToken/admin/deposit_minter_capability.cdc
    //
    pub fun createNewMinter(): @Minter {
        emit MinterCreated()
        return <- create Minter()
    }

}
```

`Minter` `resource`包含对应的 `mintTokens function`可以铸造新的FUSD，并将新铸造的FUSD数量累加到总供应量（ `totalSupply`）中。mintTokens function利用 `pre condition`确保amount变量为正数，并创建返回一个 `FUSD.Vault resource`。

```swift
// Minter
//
// Resource object that can mint new tokens.
// The admin stores this and passes it to the minter account as a capability wrapper resource.
//
pub resource Minter {

    // mintTokens
    //
    // Function that mints new tokens, adds them to the total supply,
    // and returns them to the calling context.
    //
    pub fun mintTokens(amount: UFix64): @FUSD.Vault {
        pre {
            amount > 0.0: "Amount minted must be greater than zero"
        }
        FUSD.totalSupply = FUSD.totalSupply + amount
        emit TokensMinted(amount: amount)
        return <-create Vault(balance: amount)
    }

}
```

通过上述代码， `AdminAccount`可以自己通过创建 `Minter Resource`铸造新的FUSD。

不过，正如我们可以看到FUSD合约中并没有为 `AdminAccount`创建Vault，因此我们需要单独用另外一个transaction创建对应的FUSD的Vault，同时如果用其他账户做测试的话，也需要对应的其他账户发送这个transaction用以创建Vault。

```bash
import FungibleToken from 0xADDRESS
import FUSD from 0xADDRESS

transaction {

    prepare(signer: AuthAccount) {

        // It's OK if the account already has a Vault, but we don't want to replace it
        if(signer.borrow<&FUSD.Vault>(from: /storage/fusdVault) != nil) {
            return
        }
        
        // Create a new FUSD Vault and put it in storage
        signer.save(<-FUSD.createEmptyVault(), to: /storage/fusdVault)

        // Create a public capability to the Vault that only exposes
        // the deposit function through the Receiver interface
        signer.link<&FUSD.Vault{FungibleToken.Receiver}>(
            /public/fusdReceiver,
            target: /storage/fusdVault
        )

        // Create a public capability to the Vault that only exposes
        // the balance field through the Balance interface
        signer.link<&FUSD.Vault{FungibleToken.Balance}>(
            /public/fusdBalance,
            target: /storage/fusdVault
        )
    }
}
```

接着我们可以用 `AdminAccount`创建Minter Resource以铸造新的FUSD，范例transaction如下。

此范例中，创建的minter resource最后是被destroy了。但也可以把这个resource存储在AdminAccount中，不需要destroy。

```swift
import FungibleToken from 0xADDRESS
import FUSD from 0xADDRESS

transaction(recipientAddress: Address, amount: UFix64) {

    let tokenReceiver: &{FungibleToken.Receiver}

    prepare(adminAccount: AuthAccount) {

      self.tokenReceiver = getAccount(recipientAddress)
          .getCapability(/public/fusdReceiver)!
          .borrow<&{FungibleToken.Receiver}>()
          ?? panic("Unable to borrow receiver reference")

      // Create a reference to the admin resource in storage.
      let tokenAdmin = adminAccount.borrow<&FUSD.Administrator>(from: FUSD.AdminStoragePath)
          ?? panic("Could not borrow a reference to the admin resource")

      // Create a new minter resource and a private link to a capability for it in the admin's storage.
      let minter <- tokenAdmin.createNewMinter()

      let mintedVault <- minter.mintTokens(amount: amount)
      self.tokenReceiver.deposit(from: <-mintedVault)

      destroy minter

    }

}
```

flow cli 范例:

```bash
flow transactions send ./transactions/fusd_direct_mint.cdc --signer ADMIN_ACCOUNT --arg Address:0x01cf0e2f2f715450 --arg UFix64:10.0
```

是不是也能通过多签(multi-signer)把这个minter resource直接授予别人？ 还没测试过。

初次此外， `AdminAccount`还可以通过 `capability`授予其他账户同样mint FUSD的权限。

大致步骤如下：

新Minter账号调用FUSD合约中的 `createMinterProxy()` function创建 `MinterProxy resource` 并存储在自己的 `storage`中，然后再把存储有 `MinterProxy resource`的 `storage` link 到 `public` 以便 `AdminAccount`调用

—> `AdminAccount`调用自己的 `Administrator` resource创建 `Minter resource`并存储到`storage`中，然后link到 `private` 以创建 `minterCapability`

—> AdminAccount通过新Minter账号中的 `public` `MinterProxy resource`，调用MinterProxy resource中 `setMinterCapability()`，授予新Minter账户铸造FUSD的权利

—> 新Minter账户通过调用自己 `MinterProxy resource`中的mintTokens() function铸造新的FUSD

FUSD合约相关代码如下：

```swift
pub resource interface MinterProxyPublic {
    pub fun setMinterCapability(cap: Capability<&Minter>)
}

// MinterProxy
//
// Resource object holding a capability that can be used to mint new tokens.
// The resource that this capability represents can be deleted by the admin
// in order to unilaterally revoke minting capability if needed.

pub resource MinterProxy: MinterProxyPublic {

    // access(self) so nobody else can copy the capability and use it.
    access(self) var minterCapability: Capability<&Minter>?

    // Anyone can call this, but only the admin can create Minter capabilities,
    // so the type system constrains this to being called by the admin.
    pub fun setMinterCapability(cap: Capability<&Minter>) {
        self.minterCapability = cap
    }

    pub fun mintTokens(amount: UFix64): @FUSD.Vault {
        return <- self.minterCapability!
        .borrow()!
        .mintTokens(amount:amount)
    }

    init() {
        self.minterCapability = nil
    }

}

// createMinterProxy
//
// Function that creates a MinterProxy.
// Anyone can call this, but the MinterProxy cannot mint without a Minter capability,
// and only the admin can provide that.
//
pub fun createMinterProxy(): @MinterProxy {
    return <- create MinterProxy()
}
```

下面依次是需要用到的transaction代码。

用新Minter账户创建MinterProxy

```swift
import FUSD from 0xADDRESS

transaction {

    prepare(adminAccount: AuthAccount) {

        let minter_proxy <- FUSD.createMinterProxy()
        adminAccount.save(<- minter_proxy, to: FUSD.MinterProxyStoragePath)

        adminAccount.link<&FUSD.MinterProxy{FUSD.MinterProxyPublic}>(
            FUSD.MinterProxyPublicPath,
            target: FUSD.MinterProxyStoragePath
        ) ?? panic("Could not link minter proxy")
    }
}
```

```swift
flow transactions send ./transactions/fusd_initialize_minter.cdc --signer new_minter_account
```

接下来这个transaction要由 `AdminAccount`发起，作用是创建 `Minter Resource`，存储并链接到 `AdminAccount`的 `private storage`中，最后再通过新Minter账户中的存储在public storage中的 `MinterProxy Resource`，调用 `setMinterCapability()`，将AdminAccount的capability链接进去。

```swift
import FUSD from 0xADDRESS

transaction(newMinterAccount: Address) {

    let resourceStoragePath: StoragePath
    let capabilityPrivatePath: CapabilityPath
    let minterCapability: Capability<&FUSD.Minter>

    prepare(adminAccount: AuthAccount) {

        // These paths must be unique within the FUSD contract account's storage
        self.resourceStoragePath = /storage/minter_01
        self.capabilityPrivatePath = /private/minter_01

        // Create a reference to the admin resource in storage.
        let tokenAdmin = adminAccount.borrow<&FUSD.Administrator>(from: FUSD.AdminStoragePath)
            ?? panic("Could not borrow a reference to the admin resource")

        // Create a new minter resource and a private link to a capability for it in the admin's storage.
        let minter <- tokenAdmin.createNewMinter()
        adminAccount.save(<- minter, to: self.resourceStoragePath)
        self.minterCapability = adminAccount.link<&FUSD.Minter>(
            self.capabilityPrivatePath,
            target: self.resourceStoragePath
        ) ?? panic("Could not link minter")

    }

    execute {

        // This is the account that the capability will be given to
        let minterAccount = getAccount(newMinterAccount)

        let capabilityReceiver = minterAccount.getCapability
            <&FUSD.MinterProxy{FUSD.MinterProxyPublic}>
            (FUSD.MinterProxyPublicPath)!
            .borrow() ?? panic("Could not borrow capability receiver reference")

        capabilityReceiver.setMinterCapability(cap: self.minterCapability)

    }

}
```

```bash
flow transactions send ./transactions/fusd_admin_create_minter.cdc --signer ADMIN_ACCOUNT --arg Address:0x01cf0e2f2f715450
```

之后，新Minter账户就可以通过下面这个transaction铸造新的FUSD。

```swift
import FungibleToken from 0xADDRESS
import FUSD from 0xADDRESS

transaction(recipientAddress: Address, amount: UFix64) {
    let tokenMinter: &FUSD.MinterProxy
    let tokenReceiver: &{FungibleToken.Receiver}

    prepare(minterAccount: AuthAccount) {
        self.tokenMinter = minterAccount
            .borrow<&FUSD.MinterProxy>(from: FUSD.MinterProxyStoragePath)
            ?? panic("No minter available")

        self.tokenReceiver = getAccount(recipientAddress)
            .getCapability(/public/fusdReceiver)!
            .borrow<&{FungibleToken.Receiver}>()
            ?? panic("Unable to borrow receiver reference")
    }

    execute {
        let mintedVault <- self.tokenMinter.mintTokens(amount: amount)

        self.tokenReceiver.deposit(from: <-mintedVault)
    }
}
```

```swift
flow transactions send ./transactions/fusd_mint_token.cdc --signer new_minter_account --arg Address:0x01cf0e2f2f715450 --arg UFix64:10.0
```

若将来AdminAccount想要取消新Minter账户铸造FUSD的权利，可以通过unlink对应的capability实现。范例transaction如下。

```bash
import FUSD from 0xADDRESS

transaction {

    let resourceStoragePath: StoragePath
    let capabilityPrivatePath: CapabilityPath

    prepare(adminAccount: AuthAccount) {

        // These paths must be unique within the FUSD contract account's storage
        self.resourceStoragePath = /storage/minter_01
        self.capabilityPrivatePath = /private/minter_01

        adminAccount.unlink(self.capabilityPrivatePath)

    }
}
```

同样的，如果你想再次授予新minter账户权利的话，可以再次link

```bash
import FUSD from 0xADDRESS

transaction {

    let resourceStoragePath: StoragePath
    let capabilityPrivatePath: CapabilityPath

    prepare(adminAccount: AuthAccount) {

        // These paths must be unique within the FUSD contract account's storage
        self.resourceStoragePath = /storage/minter_01
        self.capabilityPrivatePath = /private/minter_01

        adminAccount.link<&FUSD.Minter>(
            self.capabilityPrivatePath,
            target: self.resourceStoragePath
        ) ?? panic("Could not link minter")

    }
}
```

可以通过下面这个Script查询账户的FUSD余额

```bash
import FungibleToken from 0xADDRESS
import FUSD from 0xADDRESS

pub fun main(address: Address): UFix64 {
    let account = getAccount(address)

    let vaultRef = account.getCapability(/public/fusdBalance)!
        .borrow<&FUSD.Vault{FungibleToken.Balance}>()
        ?? panic("Could not borrow Balance reference to the Vault")

    return vaultRef.balance
}
```

```bash
flow scripts execute ./scripts/check_fusd_balance.cdc --arg Address:0x01cf0e2f2f715450
```

### 一些想法

在我的一个应用场景中，我需要授予指定用户可以自行生成NFT的权利，并且我还希望我能够随时取消他们的mint的权利。

我有思考过比如通过多签(multi-signer)的方案，把minter的resource从admin账户转移到指定账户，但由于我们的平台并不托管用户的私钥，并且Flow-FCL貌似还不能支持多签（@Caos有指点对应的代码可以查看，我还没来得及尝试，多谢Caos！）。因此多签的方案目前还不可行。

在Flow论坛上，也有对应的讨论：

[https://forum.onflow.org/t/private-capability-revoke/1997](https://forum.onflow.org/t/private-capability-revoke/1997)

后来蒙@Caos和@Lsy指教，FUSD通过另一种方式间接的实现了这一功能。于是我开始深入研习FUSD代码，同时我发现官方的Github缺失了一部分对应的transaction，因此也就有了这篇文档。

但正如FUSD官方团队指出，FUSD的解决方案并不完美。因为，我们需要在AdminAccount中为每一个新minter账户，建造对应的storage，比如上面提高的 `minter_01`，也就是说，我们需要再单独维护一个表格用以记录对应的信息，哪个storage对应的是哪个新minter账户，并且当我们需要撤销某个账户对应的权利的时候，我们还需要额外再记录这个account的mint权利是否还有效，这无疑增加了维护工作。

如果大家有更好的解决方案，欢迎大家与我联系！

### 备注

请根据具体需要修改0xADDRESS。