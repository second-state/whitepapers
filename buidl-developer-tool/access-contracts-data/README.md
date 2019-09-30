# Data-driven DApps

The [Getting started guide](../getting-started/) showcased a number storage contract and DApp. In this section, we will use a similar smart contract, but to develop a new DApp that highlights data capabilities of the Second State platform, which supports [web3](https://github.com/second-state/web3-ss.js) for transactional data and [elastic search](https://github.com/second-state/es-ss.js) for state data. The complete source code for this example is [available here](https://github.com/second-state/buidl/tree/master/demo/data-v2).

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

Compile and deploy the smart contract via the **Compile** and **Deploy on the chain** buttons as we did in the [Getting started guide](../getting-started/). 

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
var contract = window.web3 && web3.ss && web3.ss.contract(abi);
var instance = contract && contract.at(cAddr);
window.addEventListener('web3Ready', function() {
  contract = web3.ss.contract(abi);
  instance = contract.at(cAddr);
  reload();
});

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





