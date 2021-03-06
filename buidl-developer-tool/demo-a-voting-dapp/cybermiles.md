---
description: >-
  Developers require CMT to deploy contracts. Users can access contracts without
  gas.
---

# CyberMiles

## See the dapp in action

{% page-ref page="./" %}

## **Step 1** 

Open the BUIDL IDE tool in any browser. [http://buidl.secondstate.io/cmt](http://buidl.secondstate.io/cmt)

## **Step 2** 

Open the Accounts tab and send a little CMTs to your default account. If you do not have CMT, you can ask for some from cmt@secondstate.io

{% hint style="info" %}
If you have CyberMiles' Venus wallet, you could opt to use Venus in the Providers tab. BUIDL and dapps it creates will now use the default account in Venus to make contract calls and to pay for gas.
{% endhint %}

## **Step 3** 

### 3.1 Copy and paste the following code to the contract section of BUIDL.

The smart contract is very simple. It provides the text and image url to be voted on, and keeps a record of votes. The `vote()` method is called by voters to vote thumb up or down.

```typescript
pragma solidity >= 0.4.0;

contract Vote {

    string public greeting;
    string public photoUrl;
    mapping (address => int) votes;
    uint ups;
    uint downs;

    constructor(string _greeting, string _photoUrl) public {
        greeting = _greeting;
        photoUrl = _photoUrl;
    }

    function vote (int _choice) public {
        require (votes[msg.sender] == 0);
        require (_choice == 1 || _choice == -1);
        votes[msg.sender] = _choice;
        if (_choice == 1) ups++;
        if (_choice == -1) downs++;
    }

    function getVotes () view public returns (uint, uint) {
        return (ups, downs);
    }

    function getVote (address _addr) view public returns (int) {
        return votes[_addr];
    }
}
```

{% hint style="info" %}
If you are not using Venus as the Provider, please open the Accounts tab and make sure that the default address has a little CMTs.
{% endhint %}

### 3.2 Click on Compile and you will see the following. Enter your text and image URL to be voted on, and then click on deploy on chain.

![](../../.gitbook/assets/screen-shot-2019-09-30-at-9.43.45-am.png)

The contract is now deployed on the CyberMiles blockchain, and you can call its functions directly from inside BUIDL.

![](../../.gitbook/assets/screen-shot-2019-09-30-at-9.44.42-am.png)

### 3.3 Go to the dapp section. Click on the Resources tab, and add the following as resources.

* CSS: [https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css](https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css)
* JavaScript: [https://code.jquery.com/jquery-3.4.1.min.js](https://code.jquery.com/jquery-3.4.1.min.js)

### 3.4 Next, copy and paste the following HTML code into the HTML editor.

```markup
<div class="container">
   <br/>
   <div class="jumbotron">
      <p class="lead" id="greeting"></p>
      <div id="imageDiv" style="display:none">
         <img id="image" src="" class="img-fluid img-thumbnail"/>
      </div>
      <hr/>
      <p id="votes" style="display:none">
         <span id="ups"></span> voted 👍 |
         <span id="downs"></span> voted 👎
      </p>
      <form id="form" class="form-inline" style="display:none">
         <button id="voteUp" type="button" onclick="return vote(1);" class="btn btn-secondary mb-2">👍</button>
         <button id="voteDown" type="button" onclick="return vote(-1);" class="btn btn-secondary mb-2">👎</button>
      </form>
      <div id="formSubmitted" style="display:none">Please wait 20 seconds ...</div>
      <div id="myVoteUp" style="display:none">You have already voted 👍</div>
      <div id="myVoteDown" style="display:none">You have already voted 👎</div>
   </div>
</div>
```

### 3.5 Copy and paste the following JavaScript code into the JS editor.

```javascript
var instance = null;
window.addEventListener('web3Ready', function() {
  var contract = web3.ss.contract(abi);
  instance = contract.at(cAddr);
  reload();
});

function reload() {
    instance.greeting(function (e, r) {
        $("#greeting").html(r);
    });
    instance.photoUrl(function (e, r) {
        if (!e && r) {
            $("#imageDiv").css("display", "block");
            $("#image").attr("src", r);
        }
    });
    instance.getVotes(function (e, r) {
        if (!e && (r[0] > 0 || r[1] > 0)) {
            $("#votes").css("display", "block");
            $("#ups").text(r[0]);
            $("#downs").text(r[1]);
        }
    });

    $("#form").css("display", "none");
    $("#formSubmitted").css("display", "none");
    web3.ss.getAccounts(function (e, address) {
        if (!e) {
            instance.getVote(address, function (ee, r) {
                if (r == 1) {
                    $("#myVoteUp").css("display", "block");
                } else if (r == -1) {
                    $("#myVoteDown").css("display", "block");
                } else {
                    $("#form").css("display", "block");
                }
            });
        }
    });
}

function vote (choice) {
    web3.ss.getAccounts(function (e, address) {
        if (!e) {
            $("#form").css("display", "none");
            $("#formSubmitted").css("display", "block");
            instance.vote(choice, {
                gas: 400000,
                gasPrice: 0
            }, function (ee, result) {
                if (ee) {
                    window.alert("Failed for " + address);
                }
            });
            setTimeout(function () {
                reload ();
            }, 20 * 1000);
        }
    });
    return false;
}
```

{% hint style="info" %}
On CyberMiles, we can make contract function calls with gas price set to zero. That is because CyberMiles allows gas free operations for many operations.
{% endhint %}

### 3.6 Click on Run to see the dapp in action! You can now vote thumb up or down inside BUIDL.

![](../../.gitbook/assets/screen-shot-2019-09-30-at-9.46.17-am.png)

### 3.7 Finally, you can publish the dapp. 

Just click on the Publish button and give the dapp a name. Once published, you can share the published URL to the public to vote on your issue!

{% hint style="info" %}
If the dapp user has Venus for CMT installed, the dapp will ask whether she would like to use her Venus account instead of auto-generated or imported accounts.
{% endhint %}

