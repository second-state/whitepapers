# Access contracts data

The [Getting started guide](getting-started.md) showcased a number storage contract and DApp. In this section, we will use the same smart contract, but to develop a new DApp that highlights data capabilities of the Second State platform, which supports [web3](https://github.com/second-state/web3-ss.js) for transactional data and [elastic search](https://github.com/second-state/es-ss.js) for state data. The complete source code for this example is [available here](https://gist.github.com/juntao/bb63b952119c3e4c56bd56601c9140c3).

Let’s first see how the DApp works. It displays all the storage contracts deployed on the blockchain, and then allows the user to store numbers in those contracts.

![](../.gitbook/assets/buidl-access_data-01.png)

Now, let’s review the code to see how this is done. First, under the **contract** tab, we compile and deploy the smart contract via the **Compile** and **Deploy** buttons as we did in the [Getting started guide](getting-started.md).

![](../.gitbook/assets/buidl-access_data-02.png)

Next, go to the **dapp** tab, and paste the following code into the HTML tab. The HTML code demonstrates how to use the bootstrap 4 CSS framework. If you have additional CSS style rules for this page, you can put them in the CSS tab.

```text
<!doctype html>
<html lang="en">
  <head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    
    <!-- jQuery first, then Popper.js, then Bootstrap JS -->
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
    
    <title>Data Stores</title>
  </head>
  <body>
    <div class="container">
        <p><button id="create" class="btn btn-primary" onclick="create(this)">Create new storage</button></p>
        <table class="table">
            <thead>
                <tr>
                    <th scope="col">Block #</th>
                    <th scope="col">Data</th>
                    <th scope="col"></th>
                </tr>
            </thead>
            <tbody id="tbody">
            </tbody>
        </table>
    </div>

  </body>
</html>
```

The HTML renders a button to create new storage contracts, as well as a table that lists all the contracts. The table is initially empty, and will be filled by the JavaScript, which we see next.

The `reload()` function in the JS code below calls the elastic search API to get all contracts with the ABI from the blockchain. It then constructs a table body to display those contracts. Notice that the current state, ie the stored number, of each contract is also contained in the search result. We can simply display this information without having to interact with the slower blockchain nodes.

```text
function reload () {
  document.querySelector("#create").innerHTML = "Create new storage";
  document.querySelector("#tbody").innerHTML = "";
  var tbodyInner = "";
  esss.shaAbi(JSON.stringify(abi)).then((shaResult) => {
    var sha = JSON.parse(shaResult).abiSha3;
    esss.searchUsingAbi(sha).then((searchResult) => {
      var items = JSON.parse(searchResult);
      items.sort(compareItem);
      items.forEach(function(item) {
        tbodyInner = tbodyInner + 
          "<tr><td>" + item.blockNumber + 
          "</td><td>" + item.functionData.get + 
          "</td><td><button class='btn btn-info' onclick='setData(this)' id='" + item.contractAddress + "'>Set</button></td></tr>";
      }); // end of JSON iterator

      document.querySelector("#tbody").innerHTML = tbodyInner;
    });
  });
```

The **Set Data** buttons in the table trigger the `setData()` JS function, which in turn calls the contract’s `set()` function via web3.

```text
function setData (element) {
  console.log("Set Data " + element.id);
  instance = contract.at(element.id);
  var n = window.prompt("Input a number:");
  n && instance.set(n);
  element.innerHTML = "Wait ..."
  setTimeout(function () {
    reload();
  }, 5 * 1000);
}
```

The **Create new storage** button triggers the `create()` JS function to create a new contract on the blockchain. It works as follows.

```text
function create (element) {
  element.innerHTML = "Wait ...";
  var data = '0x' + contract.new.getData({data:bytecode});
  contract.new({
    data: data
  }, function (ee, i) {
    if (ee) {
      console.log("Error creating contract " + ee);
    } else {
      // May take a few seconds for i.address to return
      setTimeout(function () {
        reload();
      }, 5 * 1000);
    }
  });
}
```

In this section, we demonstrated how to use [web3](https://github.com/second-state/web3-ss.js) and [elastic search](https://github.com/second-state/es-ss.js) APIs together to build high performance and data driven DApps. The [FairPlay DApp](https://www.fairplaydapp.com/) is a successful real world example based on [this approach](../white-papers/fairplay-a-new-type-of-dapp.md).





