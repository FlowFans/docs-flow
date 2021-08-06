---
description: '@Lsy'
---

# Flow 测试框架的基本使用

## Flow Network 的 JavaScript 测试框架

 该存储库包含实用方法，这些方法与测试库（如 ）结合使用`Jest`，可在使用 Cadence 构建 Flow dapp 时提高您的工作效率。



### 基本要求

*  假设您在 Node 环境下使用这个框架。您至少需要版本**12.0.0**
*  在后台运行 Flow Emulator。您可以将它与 Flow CLI 一起安装。 有关说明，请参阅[安装 Flow CLI](https://docs.onflow.org/flow-cli/install)。
*  如果您已经安装了它，请在终端中运行`flow init`以创建`flow.json`配置文件。然后用 启动模拟器`flow emulator -v`。

### 在 Playground 编写 Cadence

每个 Playground 项目都能够将`export`其内容作为一组带有 Cadence 模板代码和“开箱即用”基本测试环境的文件。

如果要使用此功能：

![](../../.gitbook/assets/image%20%282%29.png)

* 按右上角的“Export”按钮
* 选择项目的名称 - 或保留自动生成的版本
* 在弹出窗口中按“Export”按钮

Playground 将为您创建一个`zip`文件，您可以将其保存在任何您喜欢的地方。

### 安装测试环境

Playground 导出的 Zip 中包含 `cadence` 目录和  `test` 目录，

安装相关依赖，你可以在命令行中执行如下命令：

```text
npm install flow-js-testing jest @babel/core @babel/preset-env babel-jest @onflow/types
```

或者直接进入 `test` 目录下运行：

```text
yarn install

// 初始化项目  生成 flow.json 文件
yarn init-flow

// 启动本地模拟器 
yarn start-emulator
```

更多详细的API ： [https://github.com/onflow/flow-js-testing/blob/master/docs/api.md](https://github.com/onflow/flow-js-testing/blob/master/docs/api.md)



