---
description: 'Better Ethereum contract abstraction, for Node and the browser.'
---

# truffle-contract

**安装**

```text
$ npm install truffle-contract
```

**特征**

* 同步交易可以实现更好的控制流程（即，在您确保已经开采之前，交易将无法完成）。
* 承诺。没有更多的回调。适用于`ES6`和`async/await`。
* 交易的默认值，如`from`地址或`gas`。
* 返回每个同步事务的日志，事务接收和事务哈希。

**用法**

首先，设置一个新的web3提供程序实例并初始化您的合约`require("truffle-contract")`。该`contract`函数的输入是由[truffle-contract-schema](https://github.com/trufflesuite/truffle-contract-schema)定义的JSON blob 。此JSON blob的结构可以传递给所有与truffle相关的项目。

```text
var provider = new Web3.providers.HttpProvider("http://localhost:8545");
var contract = require("truffle-contract");

var MyContract = contract({
  abi: ...,
  unlinked_binary: ...,
  address: ..., // optional
  // many more
})
MyContract.setProvider(provider);
```

您现在可以访问以下`MyContract`许多功能，以及许多其他功能：

* `at()`：创建一个`MyContract`代表您在特定地址的合约的实例。
* `deployed()`：创建一个实例`MyContract`，表示由其管理的默认地址`MyContract`。
* `new()`：将此合同的新版本部署到网络，获取其中的实例`MyContract`表示新部署的实例。

每个实例都绑定到以太坊网络上的特定地址，每个实例都具有从Javascript函数到契约函数的1对1映射。例如，如果您的Solidity合约定义了一个函数`someFunction(uint value) {}`（solidity），那么您可以在网络上执行该函数，如下所示：

```text
var deployed;
MyContract.deployed().then(function(instance) {
  var deployed = instance;
  return instance.someFunction(5);
}).then(function(result) {
  // Do something with the result or continue with more transactions.
});
```

**浏览器用法**

在您的`head`元素中，包括Web3，然后包含松露合同：

```text
<script type="text/javascript" src="./path/to/web3.min.js"></script>
<script type="text/javascript" src="./dist/truffle-contract.min.js"></script>
```

或者，您可以使用非缩小版本以便于调试。

有了这种用法，`truffle-contract`将通过`TruffleContract`对象提供：

```text
var MyContract = TruffleContract(...);
```

**完整的例子**

让我们使用[Dapps For Beginners](https://dappsforbeginners.wordpress.com/tutorials/your-first-dapp/)`truffle-contract`的示例合同。在这种情况下，抽象已被[truffle-artifactor](https://github.com/trufflesuite/truffle-artifactor)保存到文件中：`.sol.js`

```text
// Require the package that was previosly saved by truffle-artifactor
var MetaCoin = require("./path/to/MetaCoin.sol.js");

// Remember to set the Web3 provider (see above).
MetaCoin.setProvider(provider);

// In this scenario, two users will send MetaCoin back and forth, showing
// how truffle-contract allows for easy control flow.
var account_one = "5b42bd01ff...";
var account_two = "e1fd0d4a52...";

// Note our MetaCoin contract exists at a specific address.
var contract_address = "8e2e2cf785...";
var coin;

MetaCoin.at(contract_address).then(function(instance) {
  coin = instance;

  // Make a transaction that calls the function `sendCoin`, sending 3 MetaCoin
  // to the account listed as account_two.
  return coin.sendCoin(account_two, 3, {from: account_one});
}).then(function(result) {
  // This code block will not be executed until truffle-contract has verified
  // the transaction has been processed and it is included in a mined block.
  // truffle-contract will error if the transaction hasn't been processed in 120 seconds.

  // Since we're using promises, we can return a promise for a call that will
  // check account two's balance.
  return coin.balances.call(account_two);
}).then(function(balance_of_account_two) {
  alert("Balance of account two is " + balance_of_account_two + "!"); // => 3

  // But maybe too much was sent. Let's send some back.
  // Like before, will create a transaction that returns a promise, where
  // the callback won't be executed until the transaction has been processed.
  return coin.sendCoin(account_one, 1.5, {from: account_two});
}).then(function(result) {
  // Again, get the balance of account two
  return coin.balances.call(account_two)
}).then(function(balance_of_account_two) {
  alert("Balance of account two is " + balance_of_account_two + "!") // => 1.5
}).catch(function(err) {
  // Easily catch all errors along the whole execution.
  alert("ERROR! " + err.message);
});
```

### API

您需要注意两个API。一个是静态合同抽象API，另一个是合同实例API。Abstraction API是一组存在于所有合同抽象中的函数，这些函数存在于抽象本身（即`MyContract.at()`）。相反，Instance API是可用于合同实例的API - 即表示网络上特定合同的抽象 - 并且API是根据Solidity源文件中可用的函数动态创建的。

**合同抽象API**

每个合同抽象 - `MyContract`在上面的例子中 - 都有以下有用的功能：

**MyContract.new（\[arg1，arg2，...\]，\[tx params\]）**

此函数采用您的契约所需的任何构造函数参数，并将合同的新实例部署到网络。有一个可选的最后一个参数，您可以使用它来传递交易参数，包括来自地址，天然气限制和天然气价格的交易。此函数返回一个Promise，它在新部署的地址处解析为合同抽象的新实例。

**MyContract.at（地址）**

此函数创建一个新的合同抽象实例，表示传入地址中的合同。返回一个“可以”的对象（为了向后兼容，还不是一个实际的Promise）。在确保代码存在于指定地址后，解析为合同抽象实例。

**MyContract.deployed（）**

在其部署的地址创建表示合同的合同抽象实例。部署的地址是给予松露合同的特殊值，当设置时，在内部保存地址，以便可以从所使用的给定以太网网络推断部署的地址。这允许您编写引用特定已部署合同的代码，而无需自己管理这些地址。就像`at()`，`deployed()`可以，并且在确保代码存在于该位置并且该地址存在于正在使用的网络上之后，将解析为表示已部署合同的合同抽象实例。

**MyContract.link（实例）**

将合同抽象实例表示的库链接到MyContract。必须首先部署库并设置其已部署的地址。名称和部署地址将从合同抽象实例中推断出来。使用此形式时`MyContract.link()`，MyContract将使用所有链接库的事件，并且能够报告在事务结果期间发生的那些事件。

库可以多次链接，并覆盖以前的链接。

注意：此方法有两种其他形式，但建议使用此表单。

**MyContract.link（姓名，地址）**

将具有特定名称和地址的库链接到MyContract。使用此表单不会使用库的事件。

**MyContract.link（对象）**

将Object表示的多个库链接到MyContract。键必须是表示库名称的字符串，值必须是表示地址的字符串。如上所述，使用此表单不会消耗库的事件。

**MyContract.networks（）**

查看此合同抽象已设置为表示的网络ID列表。

**MyContract.setProvider（供应商）**

设置此合约抽象将用于进行事务的web3提供程序。

**MyContract.setNetwork（NETWORK\_ID）**

设置MyContract当前表示的网络。

**MyContract.hasNetwork（NETWORK\_ID）**

返回一个布尔值，表示此合约抽象是否设置为表示特定网络。

**MyContract.defaults（\[new\_defaults\]）**

获取并选择为从此抽象创建的所有实例设置事务默认值。如果在没有任何参数的情况下调用它，它将只返回表示当前默认值的Object。如果传递了一个Object，则会设置新的默认值。可以设置的默认事务值示例如下：

```text
MyContract.defaults({
  from: ...,
  gas: ...,
  gasPrice: ...,
  value: ...
})
```

`from`例如，当您有一个合同抽象来表示一个用户（即一个地址）时，设置一个默认地址很有用。

**MyContract.clone（NETWORK\_ID）**

克隆合同抽象以获取管理相同合同工件的另一个对象，但使用不同的`network_id`。如果您想在不同的网络上管理相同的合同，这非常有用。使用此功能时，请不要忘记以后设置正确的提供程序。

```text
var MyOtherContract = MyContract.clone(1337);
```

**合同实例API**

每个合同实例根据源Solidity合同而不同，并且API是动态创建的。出于本文档的目的，让我们使用以下Solidity源代码：

```text
contract MyContract {
  uint public value;
  event ValueSet(uint val);
  function setValue(uint val) {
    value = val;
    ValueSet(value);
  }
  function getValue() constant returns (uint) {
    return value;
  }
}
```

从JavaScript的角度来看，本合同具有三个功能：`setValue`，`getValue`和`value`。这是因为`value`是公共的并自动为它创建一个getter函数。

**通过合同功能进行交易**

当我们打电话时`setValue()`，这会创建一个交易。来自Javascript：

```text
instance.setValue(5).then(function(result) {
  // result object contains import information about the transaction
  console.log("Value was set to", result.logs[0].args.val);
});
```

返回的结果对象如下所示：

```text
{
  tx: "0x6cb0bbb6466b342ed7bc4a9816f1da8b92db1ccf197c3f91914fc2c721072ebd",
  receipt: {
    // The return value from web3.eth.getTransactionReceipt(hash)
    // See https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgettransactionreceipt
  },
  logs: [
    {
      address: "0x13274fe19c0178208bcbee397af8167a7be27f6f",
      args: {
        val: BigNumber(5),
      },
      blockHash: "0x2f0700b5d039c6ea7cdcca4309a175f97826322beb49aca891bf6ea82ce019e6",
      blockNumber: 40,
      event: "ValueSet",
      logIndex: 0,
      transactionHash: "0x6cb0bbb6466b342ed7bc4a9816f1da8b92db1ccf197c3f91914fc2c721072ebd",
      transactionIndex: 0,
      type:"mined",
    },
  ],
}
```

请注意，如果事务中正在执行的函数具有返回值，则不会在此结果中获得该返回值。您必须改为使用事件（例如`ValueSet`）并在`logs`数组中查找结果。

**明确地拨打电话而不是交易**

我们可以`setValue()`通过显式使用`.call`以下方式调用而不创建事务

```text
instance.setValue.call(5).then(...);
```

这在这种情况下不是很有用，因为`setValue()`设置了东西，我们传递的值将不会保存，因为我们没有创建事务。

**呼唤吸气剂**

但是，我们可以使用，_获取_值。呼叫始终是免费的，不需要任何以太网，因此它们适用于调用从区块链读取数据的函数：`getValue().call()`

```text
instance.getValue.call().then(function(val) {
  // val reprsents the `value` storage object in the solidity contract
  // since the contract returns that value.
});
```

更有帮助的是，我们_甚至不需要_`.call`在函数被标记为时使用`constant`，因为它`truffle-contract`会自动知道该函数只能通过调用进行交互：

```text
instance.getValue().then(function(val) {
  // val reprsents the `value` storage object in the solidity contract
  // since the contract returns that value.
});
```

**处理交易结果**

当您进行交易时，您将获得一个`result`对象，该对象为您提供有关交易的大量信息。您将获得事务has（`result.tx`），已解码事件（也称为日志; `result.logs`）和事务收据（`result.receipt`）。在下面的示例中，您将收到`ValueSet()`事件，因为您使用以下`setValue()`函数触发了事件：

```text
instance.setValue(5).then(function(result) {
  // result.tx => transaction hash, string
  // result.logs => array of trigger events (1 item in this case)
  // result.receipt => receipt object
});
```

**发送以太/触发回退功能**

您可以通过向此函数发送事务来触发回退功能：

```text
instance.sendTransaction({...}).then(function(result) {
  // Same result object as above.
});
```

这与所有可用的合同实例函数一样被默认，并且具有与`web3.eth.sendTransaction`没有回调的API相同的API 。该`to`值将自动填写给您。

如果您只想将以太币发送给合同，可以使用速记：

```text
instance.send(web3.toWei(1, "ether")).then(function(result) {
  // Same result object as above.
});
```

**估算天然气的使用量**

运行此功能以估算燃气使用情况：

```text
instance.setValue.estimateGas(5).then(function(result) {
  // result => estimated gas for this transaction
});
```

### 测试

该软件包是将EtherPudding分解为多个模块的结果。测试目前存在于[松露神器内，](https://github.com/trufflesuite/truffle-artifactor)但很快就会移到这里。

