The main instruction is [here](https://medium.com/taipei-ethereum-meetup/a-complete-guide-on-building-a-smart-contract-on-a-private-net-in-ethereum-726851c7c044)
But for versions >=1.6 part about compiling smart contract is incorrect. Here is correct steps:

0. check this [comment and issue](https://github.com/ethereum/go-ethereum/issues/14850#issuecomment-319751265)
1. start rpc server `geth --rpc --rpcaddr 127.0.0.1 --rpcport 8545 --dev --datadir privchain`
2. attach to server `geth attach ipc://[FULL_PATH]/privchain/geth.ipc`
3. create contract.sol with your code `contract test { function multiply(uint a) returns(uint d){return a * 7;}}`
4. run `solc --optimize --combined-json abi,bin,interface contract.sol > contract.js`
5. edit contract.js - add `var compilerOutput=` at start:
```var compilerOutput={"contracts":{"contract.sol:test":{"abi":"[{\"constant\":false,\"inputs\":[{\"name\":\"a\",\"type\":\"uint256\"}],\"name\":\"multiply\",\"outputs\":[{\"name\":\"d\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"}]","bin":"60606040523415600e57600080fd5b5b60978061001d6000396000f300606060405263ffffffff7c0100000000000000000000000000000000000000000000000000000000600035041663c6888fa18114603c575b600080fd5b3415604657600080fd5b604f6004356061565b60405190815260200160405180910390f35b600781025b9190505600a165627a7a72305820cf8e0a51d9910e28b654c622345617705d3f782936d37108f340cbe0fc1a3de80029"}},"version":"0.4.16+commit.d7661dd9.Darwin.appleclang"}```
6. in console eth `loadScript("FULL_PATH/contract.js")`
7. `var testContract = web3.eth.contract(JSON.parse(compilerOutput.contracts["contract.sol:test"].abi));`
8. `personal.newAccount('12345')`
9. `miner.setEtherbase(eth.accounts[eth.accounts.length-1])`
10. `account1 = web3.eth.coinbase`
11. `miner.start()`
12. `web3.eth.getBalance(account1)`
13. `personal.unlockAccount(account1, '12345');`
14. ```var contractDeployed = testContract.new({data: "0x" + compilerOutput.contracts["contract.sol:test"].bin, gas: 1000000, from: account1}, function(e, contract){if(!e){if(!contract.address){console.log("Contract transaction send: Transaction Hash: "+contract.transactionHash+" waiting to be mined...");}else{console.log("Contract mined! Address: "+contract.address);console.log(contract);}}else{console.log(e)}});```
Expected output: ```Contract transaction send: Transaction Hash: 0x4df25e70d41bdc92631fdc20b48474126ef107f91e8eaf33c99e012feb365821 waiting to be mined...
Contract mined! Address: 0x392e608e31c264f6c30cf3848e74e476e602e5a2
```
15. `eth.getCode(contractDeployed.address)`
16. `contractDeployed.multiply.call(6)`
