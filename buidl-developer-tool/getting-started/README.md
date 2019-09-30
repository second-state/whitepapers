---
description: >-
  The BUIDL online IDE can create complete DApps (smart contracts + web3 +
  ElasticSearch + JS + HTML / CSS) in minutes without installing any software.
---

# Getting started

In this section, we will walk through the default example that comes with the [BUIDL](http://buidl.secondstate.io/) IDE. The complete source code for this example is [available here](https://github.com/second-state/buidl/tree/master/demo/default).

**Access the BUIDL IDE from your browser:** [**https://buidl.secondstate.io/**](https://buidl.secondstate.io/)\*\*\*\*

{% embed url="https://www.youtube.com/watch?v=K8gMUeOwz1s" caption="Watch a 4-min video on how to create your first DApp in BUIDL" %}

{% hint style="info" %}
BUIDL works with the [Second State DevChain](../../devchain/getting-started.md) by default. It could also work with any [blockchain started by the Second BaaS](../working-with-baas.md) service, as well as any [Ethereum compatible blockchains]().
{% endhint %}

#### Step 1: Create and deploy a simple Solidity smart contract

Load [BUIDL](http://buidl.secondstate.io/) in your web browser. You will see a simple smart contract already in the online editor window.

![](../../.gitbook/assets/buidl-getting_started-01.png)

The contract simply allows you to store a number on the blockchain. You can view or update the stored number by calling its functions `get()` and `set()`.

Click on the **Compile** button to compile the contract. A side bar will open to show you the compiled ABI and bytecode of the contract.

![](../../.gitbook/assets/buidl-getting_started-02.png)

Next, you can press the **Deploy to the chain** button on the left panel to instantiate and deploy the contract to the [Second State DevChain](../../smart-contracts-search-engine/getting-started.md). You can interact with deployed contracts by calling its public methods from inside [BUIDL](http://buidl.secondstate.io/) -- you can **set** its value and click on the **Transact** button to save the value onto the blockchain, and then click on the **Call** button to see the value in the **LOG** panel.

![](../../.gitbook/assets/buidl-getting_started-03.png)

#### Step 2: Create a UI in HTML

Once deployed, click on the **dapp** button on the left bar to work on your DApp.

![](../../.gitbook/assets/buidl-getting_started-04.png)

The HTML tab above shows a simple HTML page with two buttons.

#### Step 3: Create JS script to interact with the smart contract

Next, go to the JS tab. It shows JavaScript on how to interact with the smart contract.

![](../../.gitbook/assets/screen-shot-2019-09-30-at-2.25.50-pm.png)

The JS has four sections. The first section is `Don't modify` as it is populated by the BUIDL tool itself. It contains information about the contract you just deployed via BUIDL.

The second section shows you the boilerplate to instantiate the `contract` and `instance` objects using data from the first section.

```javascript
var contract = window.web3 && web3.ss && web3.ss.contract(abi);
var instance = contract && contract.at(cAddr);
window.addEventListener('web3Ready', function() {
  contract = web3.ss.contract(abi);
  instance = contract.at(cAddr);
});
```

The third section is the event handler for the **Set Data** button. It shows how to call the smart contract's `set()` function in a transaction from JavaScript.

```javascript
document.querySelector("#s").addEventListener("click", function() {
  var n = window.prompt("Input the number:");
  n && instance.set(n);
});
```

The last section is the event handler for the **Get Data** button. It calls the smart contract’s `get()` function and displays the result.

```javascript
document.querySelector("#g").addEventListener("click", function() {
  console.log(instance.get().toString());
});
```

#### Step 4: Run the DApp

Finally, click on the **Run** button to run the DApp. You will see the DApp UI in the right panel. You can click on the **Set Data** button to store a number, and **Get Data** button to retrieve the stored number.

![](../../.gitbook/assets/buidl-getting_started-06.png)

Congratulations. You now have a complete DApp deployed on a public blockchain! 

Next, you could explore how to develop more complex DApps on BUIDL, such as [data driven DApps](../access-contracts-data/) or [rules-based DApps](../rule-based-smart-contract.md).



