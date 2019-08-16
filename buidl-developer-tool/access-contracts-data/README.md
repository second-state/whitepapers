# Data-driven DApps

The [Getting started guide](../getting-started.md) showcased a number storage contract and DApp. In this section, we will use a similar smart contract, but to develop a new DApp that highlights data capabilities of the Second State platform, which supports [web3](https://github.com/second-state/web3-ss.js) for transactional data and [elastic search](https://github.com/second-state/es-ss.js) for state data. The complete source code for this example is [available here](https://github.com/second-state/buidl/tree/master/demo/data-v2).

**Access the BUIDL IDE from your browser:** [**https://buidl.secondstate.io/**](https://buidl.secondstate.io/)\*\*\*\*

Let’s first see how the DApp works. It displays a number of `AccountBalanceDemo` contracts deployed on the blockchain. Each of those contracts stores a number that can be changed by the user. The search engine tracks and displays the tally of those numbers inside the contracts in real time.

{% embed url="https://www.youtube.com/watch?v=6sQHDUkbz6k" caption="Watch a video on how to create and run the DApp" %}

Now, let’s review the code to see how this is done.

#### Step 1: Copy and paste the following code into contract tab

```typescript
pragma solidity >=0.4.0 <0.6.0;

contract AccountBalanceDemo {

    string accountName;
    uint accountBalance;

    constructor(string _accountName) public {
        accountName = _accountName;
    }

    function setAccountBalance(uint _accountBalance) public {
        accountBalance = _accountBalance;
    }

    function getAccountName() public view returns(string) {
        return accountName;
    }

    function getAccountBalance() public view returns(uint) {
        return accountBalance;
    }
}
```

Compile and deploy the smart contract via the **Compile** and **Deploy on the chain** buttons as we did in the [Getting started guide](../getting-started.md). 

> Make sure that you give the account a name in the `_accountName` field above the **Deploy on the chain** button.

#### Step 2: Copy and paste the follow HTML code into the dapp -&gt; HTML tab

```markup
<!doctype html>
<html lang="en">
   <head>
      <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
      <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
      <title>Data Stores</title>
   </head>
   <body>
      <div class="container">
         <p>This page shows a list of individual accounts and their individual balances.</p>
         <p>Each account entity (each instantiation of the smart contract) is not aware of the other accounts, or their balances</p>
         <p>This page demonstrates how the smart contract search engine can provide the sum total of all accounts combined.</p>
         <br />
         <b>Sum total of all accounts</b>
         <table class="table">
            <thead>
               <tr><th scope="col">Total</th></tr>
            </thead>
            <tbody id="totalBody"></tbody>
         </table>
         <br />
         <b>Name and balance of individual accounts</b>
         <table class="table">
            <thead>
               <tr>
                  <th scope="col">Account Name</th>
                  <th scope="col">Account Balance</th>
                  <th scope="col"></th>
               </tr>
            </thead>
            <tbody id="individualBody"></tbody>
         </table>
      </div>
   </body>
</html>
```

The HTML code demonstrates how to use the bootstrap 4 CSS framework. If you have additional CSS style rules for this page, you can put them in the CSS tab.

The HTML renders a table that lists all the `AccountBalanceDemo` contracts, as well as the total tally from those contracts.

#### Step 3: Copy and paste the following into the dapp -&gt; JS tab

```javascript
/* Don't modify */
var abi = [{"constant":false,"inputs":[{"name":"_accountBalance","type":"uint256"}],"name":"setAccountBalance","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"getAccountName","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getAccountBalance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_accountName","type":"string"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"}];
var bytecode = '608060405234801561001057600080fd5b5060405161032a38038061032a833981018060405281019080805182019291905050508060009080519060200190610049929190610050565b50506100f5565b82805460018160011615610100020d166002900490600052602060002090601f016020900481019282601f1061009157805160ff19168380011785556100bf565b828001600101855582156100bf579182015b828111156100be5782518255916020019190600101906100a3565b5b5090506100cc91906100d0565b5090565b6100f291905b808211156100ee5760008160009055506001016100d6565b5090565b90565b610226806101046000396000f300608060405260043610610057576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806308aba5aa1461005c578063638dc2b5146100895780636896fabf14610119575b600080fd5b34801561006857600080fd5b5061008760048036038101908080359060200190929190505050610144565b005b34801561009557600080fd5b5061009e61014e565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156100de5780820151818401526020810190506100c3565b50505050905090810190601f16801561010b5780820d805160018360200d6101000a0d1916815260200191505b509250505060405180910390f35b34801561012557600080fd5b5061012e6101f0565b6040518082815260200191505060405180910390f35b8060018190555050565b60606000805460018160011615610100020d166002900480601f016020809104026020016040519081016040528092919081815260200182805460018160011615610100020d166002900480156101e65780601f106101bb576101008083540402835291602001916101e6565b820191906000526020600020905b8154815290600101906020018083116101c95782900d601f168201915b5050505050905090565b60006001549050905600a165627a7a72305820fdcdd97421dfcc38167c684fb15f2a5f0f404fb0f79165df46bc3bf1c022c7ac0029';
var contract = web3.ss.contract(abi);
var instance = contract.at('0x6a0f39327527b4b590c54c9d5032046898a6db87');
/* Don't modify */

reload();

function reload() {
    document.querySelector("#totalBody").innerHTML = "";
    document.querySelector("#individualBody").innerHTML = "";
    var tbodyInner = "";
    esss.shaAbi(JSON.stringify(abi)).then((shaResult) => {
        var sha = JSON.parse(shaResult).abiSha3;
        esss.searchUsingAbi(sha).then((searchResult) => {
            var items = JSON.parse(searchResult);
            items.sort(compareItem);
            items.forEach(function(item) {
                tbodyInner = tbodyInner +
                    "<tr id='" + item.contractAddress + "'><td>" + item.functionData.getAccountName +
                    "</td><td>" + item.functionData.getAccountBalance +
                    "</td><td><button class='btn btn-info' onclick='setNumber(this)'>Update balance</button></td></tr>";
            }); // end of JSON iterator
            document.querySelector("#individualBody").innerHTML = tbodyInner;
            displayTotal();
        });
    }); // end of esss
}

function displayTotal() {
    esss.shaAbi(JSON.stringify(abi)).then((shaResult) => {
        var sha = JSON.parse(shaResult).abiSha3;
        esss.searchUsingAbi(sha).then((searchResult) => {
            var items = JSON.parse(searchResult);
            var totalBodyInner = "";
            var total = 0;
            items.forEach(function(item) {
                total = total + parseInt(item.functionData.getAccountBalance);
            });
            totalBodyInner = totalBodyInner + "<tr id='total'><td>" + total + "</tr>";
            document.querySelector("#totalBody").innerHTML = totalBodyInner;
        });
    }); // end of esss
}

function setNumber(element) {
    var tr = element.closest("tr");
    instance = contract.at(tr.id);
    var n = window.prompt("Input a number:");
    n && instance.setAccountBalance(n);
    setTimeout(function() {
        element.innerHTML = "Sending ...";
        esss.updateStateOfContractAddress(JSON.stringify(abi), instance.address).then((c2i) => {
            element.innerHTML = "Calculating ...";
            setTimeout(function() {
                reload();
            }, 5 * 1000);
        });
    }, 1 * 1000);
}

function compareItem(a, b) {
    let comparison = 0;
    if (a.functionData.getAccountName < b.functionData.getAccountName) {
        comparison = -1;
    } else if (a.functionData.getAccountName > b.functionData.getAccountName) {
        comparison = 1;
    }
    return comparison;
}
```

When the page loads, the `reload()` JS function calls the elastic search API to get all contracts with the `AccountBalanceDemo` type from the blockchain. It then constructs a table body to display those contracts. Notice that the current state, ie the stored number, of each contract is contained in the search result. We can simply display this information without having to interact with the slower blockchain nodes.

```javascript
esss.shaAbi(JSON.stringify(abi)).then((shaResult) => {
  var sha = JSON.parse(shaResult).abiSha3;
  esss.searchUsingAbi(sha).then((searchResult) => {
    var items = JSON.parse(searchResult);
    // Puts the items into the table
    displayTotal();
  });
});
```

The `displayTotal()` JS function tallies the account balances from all contracts in the search result, and displays the sum on the UI.

```javascript
function displayTotal() {
    esss.shaAbi(JSON.stringify(abi)).then((shaResult) => {
        var sha = JSON.parse(shaResult).abiSha3;
        esss.searchUsingAbi(sha).then((searchResult) => {
            var items = JSON.parse(searchResult);
            var totalBodyInner = "";
            var total = 0;
            items.forEach(function(item) {
                total = total + parseInt(item.functionData.getAccountBalance);
            });
            totalBodyInner = totalBodyInner +
                "<tr id='total'><td>" + total + "</tr>";
            document.querySelector("#totalBody").innerHTML = totalBodyInner;
        });
    }); // end of esss
}
```

The **Update balance** buttons in the table trigger the `setNumber()` JS function, which in turn call the contract’s `setAccountBalance()` function via web3. It then calls the `esss.updateStateOfContractAddress()` function to explicitly inform the search engine that this contract has changed. It then waits a short time before calling the `reload()` JS function to reload and display all contracts and the total from the search engine again. Notice that the search engine works in near real-time, as the account balance you just updated is reflected in the search results in seconds.

```javascript
function setNumber(element) {
    var tr = element.closest("tr");
    instance = contract.at(tr.id);
    var n = window.prompt("Input a number:");
    n && instance.setAccountBalance(n);
    setTimeout(function() {
        element.innerHTML = "Sending ...";
        esss.updateStateOfContractAddress(JSON.stringify(abi), instance.address).then((c2i) => {
            element.innerHTML = "Calculating ...";
            setTimeout(function() {
                reload();
            }, 5 * 1000);
        });
    }, 1 * 1000);
}
```

#### Step 4: Hit the Run button to launch the DApp

You will see the web app running inside the right panel. You can update the balance in each account, and see the total changes.

In this article, we demonstrated how to use [web3](https://github.com/second-state/web3-ss.js) and [elastic search](https://github.com/second-state/es-ss.js) APIs together to build high performance and data driven DApps. The [FairPlay DApp](https://www.fairplaydapp.com/) is a successful real world example based on [this approach](../../white-papers/fairplay-a-new-type-of-dapp.md).

If you are interested in learning more about the smart contract search engine, please read on to the next article on how to [create and index contracts](example-contract-indexing.md) programmatically. Or, you can skip ahead and learn how to write [rule-based smart contracts](../rule-based-smart-contract.md) in BUIDL.





