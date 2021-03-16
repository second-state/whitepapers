---
description: Ewasm is the next generation Ethereum virtual machine.
---

# Ethereum flavored WebAssembly \(Ewasm\)

The Ewasm is the future of Ethereum smart contracts and applications. It is highly-efficient, supported by the mainstream tech communities, and could support more programming languages on the frontend.

## Step 1 Compile your smart contract to WebAssembly bytecode

Here is a [simple ERC20 smart contract](https://github.com/second-state/oasis-ssvm-runtime/blob/ssvm/resources/erc20/erc20.sol) written in Solidity. Compiling it with the [Second State SOLL compiler](https://github.com/second-state/SOLL), you will get a [WebAssembly bytecode](https://github.com/second-state/oasis-ssvm-runtime/blob/ssvm/resources/erc20/erc20.wasm) file. Then, convert it to [HEX text](https://github.com/second-state/oasis-ssvm-runtime/blob/ssvm/resources/erc20/erc20.hex) so that we can deploy via web3 JavaScript.

* [erc20.sol](https://github.com/second-state/oasis-ssvm-runtime/blob/ssvm/resources/erc20/erc20.sol)
  * This file is an ERC20 contract written in Solidity.
* [erc20.wasm](https://github.com/second-state/oasis-ssvm-runtime/blob/ssvm/resources/erc20/erc20.wasm)
  * This file is a wasm file generate from `erc20.sol` by [SOLL](https://github.com/second-state/soll)
  * Command to generate wasm file: `soll -deploy=Normal erc20.sol`
* [erc20.hex](https://github.com/second-state/oasis-ssvm-runtime/blob/ssvm/resources/erc20/erc20.hex)
  * To deploy wasm file to our node, we need to convert `erc20.wasm` to hex.
  * Command to generate hex file: `xxd -p erc20.wasm | tr -d $'\n' > erc20.hex`

## Step 2 Deploy the contract via web3

The Node.js application [`deploy_contract.js`](http://buidl.secondstate.io/oasis-testnet/deploy_contract.js) deploys the ERC20 smart contract to the mainnet node. It uses the HEX bytecode from SOLL, as well as the private key from a known address to pay for "gas", and then initiate a coin transfer transaction.

```text
$ node deploy_contract.js

Web3 is connected.
accounts: ["0x1BbE5edc5Caabf4517e40b766D64c3DEd86822Df"]
Contract created at 0x984718904f853A004F145d133dEAb0c1dE50466B
contract.balanceOf(0x1BbE5edc5Caabf4517e40b766D64c3DEd86822Df) = 1000
```

## Step 3 Make an ERC20 coin transfer

The Node.js application [`transfer_erc20.js`](http://buidl.secondstate.io/oasis-testnet/transfer_erc20.js) makes a transfer from the ERC20 contract we just created. It transfers from the contract creator's address, which we know the private key of, to another address, and then prints the balance. The first argument is the ERC20 contract address, the second argument is the TO address that receives the ERC20 tokens, and the third argument is the amount of ERC20 tokens to transfer.

```text
$ node transfer_erc20.js 0x984718904f853A004F145d133dEAb0c1dE50466B 0x987652e1C2B3B953354A43171063499DCE16dC8f 10

Web3 is connected.
accounts: ["0x1BbE5edc5Caabf4517e40b766D64c3DEd86822Df"]
Contract instaniated at 0x984718904f853A004F145d133dEAb0c1dE50466B

Initial balances
0x1BbE5edc5Caabf4517e40b766D64c3DEd86822Df = 1000
0x987652e1C2B3B953354A43171063499DCE16dC8f = 0
Transfer 10 token from address(0x1BbE5edc5Caabf4517e40b766D64c3DEd86822Df) to address(0x987652e1C2B3B953354A43171063499DCE16dC8f)

End balances
0x1BbE5edc5Caabf4517e40b766D64c3DEd86822Df = 990
0x987652e1C2B3B953354A43171063499DCE16dC8f = 10
```

This is just a taste of the future. As Ewasm evolves, we will see many more smart contracts and dapps written in WebAssembly in the future.

