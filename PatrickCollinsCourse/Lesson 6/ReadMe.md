# Foundry Simple Storage

Foundry.toml - gives us configuration parameters while working with foundry.

## Compiling in Foundry
To compile a solidity file in foundry:
```
forge compile
```
This compiles the file and gives the output under a new directory named as ```out```.

## Deploying to a local blockchain
Foundry comes built-in with a virtual environment.

    anvil
This gives us an output for test accounts, their balances and their private keys.
You can also download Ganache and install it as it provides much more info for testing your contracts (code) on blockchain.

For any blockchain, the RPC URL is the actual HTTPs endpoint we interact with and send our API calls. Whenever we deploy a contract, send a transaction, or interact with metamask, we are essentially making an API call to that RPC endpoint.

However, a much-preferred choice is that of `Ganache`. 

> However, if you do not wish to go through the setup process. Download and run the `Ziion` OS in your virtualization software to find a pre-built environment.

## Deploying a contract
We can use forge to deploy our contract:

        forge create {ContractName} --rpc-url http://url.here:port --interactive --legacy
        forge create SimpleStorage --rpc-url http://127.0.0.1:7545 --interactive --legacy
The `--legacy` flag is used to bypass the EIP-1559 check. The `--interactive` flag is submitted so that we can submit the private key in the prompt, that wallet will be used to deploy the contract.

If you are using `anvil` we can simply use:

        forge create SimpleStorage --interactive
This will work with `anvil` as it is default to foundry.

In the directory structure, the `lib/forge-std` directory stands for forge standard library which has a bunch of helpful scripts to work with foundry. To deploy as well we can do the following:

        pragma solidity ^0.8.18;
        import {Script} from "forge-std/Script.sol";
        import {SimpleStorage} from "../src/SimpleStorage.sol";
        contract DeploySimpleStorage is Script {
            function run() external returns (SimpleStorage) {
                vm.startBroadcast();
                SimpleStorage simpleStorage = new SimpleStorage();
                vm.stopBroadcast();
                return simpleStorage;
            }
        }


`vm` is a special keyword in the `forge-std` library and is available in foundry only.
Any tx. we want to send needs to go between vm.startBroadcast() and vm.stopBroadcast().
Deploying the contract can be done just by the code between the above-mentioned code lines.

Now to run these script we use the command:

        forge script script/DeploySimpleStorage.s.sol
If we don't specify any rpc url then it runs the script on a temporary anvil chain. 

        forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545
This then deploys it on the local anvil chain (after spinning up anvil in another terminal), and gives us a new directory called 'broadcast' which gives us info about our previous deployments.

        forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545 --broadcast --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80


## Transaction:
A transaction has details like this:

        "transaction": {
        "type": "0x02",
        "from": "0x1804c8ab1f12e6bbf3894d4083f33e07309d1f38",
        "gas": "0x714e1",
        "value": "0x0",
        "data": "0x608060405234801561001057600080fd5b5061057f806100206000396000f3fe608060405234801561001057600080fd5b50600436106100575760003560e01c80632e64cec11461005c5780632ebce631146100735780636057361d146100945780636f760f41146100a95780638bab8dd5146100bc575b600080fd5b6000545b6040519081526020015b60405180910390f35b610086610081366004610248565b6100e7565b60405161006a929190610285565b6100a76100a2366004610248565b600055565b005b6100a76100b7366004610362565b61019f565b6100606100ca3660046103a7565b805160208183018101805160028252928201919093012091525481565b600181815481106100f757600080fd5b6000918252602090912060029091020180546001820180549193509061011c906103e4565b80601f0160208091040260200160405190810160405280929190818152602001828054610148906103e4565b80156101955780601f1061016a57610100808354040283529160200191610195565b820191906000526020600020905b81548152906001019060200180831161017857829003601f168201915b5050505050905082565b6040805180820190915281815260208101838152600180548082018255600091909152825160029091027fb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf68101918255915190917fb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf70190610220908261046d565b50505080600283604051610234919061052d565b908152604051908190036020019020555050565b60006020828403121561025a57600080fd5b5035919050565b60005b8381101561027c578181015183820152602001610264565b50506000910152565b82815260406020820152600082518060408401526102aa816060850160208701610261565b601f01601f1916919091016060019392505050565b634e487b7160e01b600052604160045260246000fd5b600082601f8301126102e657600080fd5b813567ffffffffffffffff80821115610301576103016102bf565b604051601f8301601f19908116603f01168101908282118183101715610329576103296102bf565b8160405283815286602085880101111561034257600080fd5b836020870160208301376000602085830101528094505050505092915050565b6000806040838503121561037557600080fd5b823567ffffffffffffffff81111561038c57600080fd5b610398858286016102d5565b95602094909401359450505050565b6000602082840312156103b957600080fd5b813567ffffffffffffffff8111156103d057600080fd5b6103dc848285016102d5565b949350505050565b600181811c908216806103f857607f821691505b60208210810361041857634e487b7160e01b600052602260045260246000fd5b50919050565b601f82111561046857600081815260208120601f850160051c810160208610156104455750805b601f850160051c820191505b8181101561046457828155600101610451565b5050505b505050565b815167ffffffffffffffff811115610487576104876102bf565b61049b8161049584546103e4565b8461041e565b602080601f8311600181146104d057600084156104b85750858301515b600019600386901b1c1916600185901b178555610464565b600085815260208120601f198616915b828110156104ff578886015182559484019460019091019084016104e0565b508582101561051d5787850151600019600388901b60f8161c191681555b5050505050600190811b01905550565b6000825161053f818460208701610261565b919091019291505056fea26469706673582212205c05838001104e8805de7d9792eb9e13e36116a987ede25a70757b149924147264736f6c63430008130033",
        "nonce": "0x2",
        "accessList": []
      },
Now some extra pieces of tx. that we might find later are:

* v,r,s components: v,r,s are the values for the transaction's signature. The v,r,s allow our private key to sign the transaction.
 The `nonce` is an important value. It counts the transaction number.

> When deploying contracts with real money, we should always use a password-encrypted keystone (something like thirdweb) or use any alternative solution wherein you do not have to write your private key in plain text.

## Interacting with Transactions

Foundry has an in-built tool called `cast` which can be used to send transactions to the contracts deployed.

    cast send --help
Requires 3 arguments: `to`, `signature`, and `argument` of the function call.

    cast send 0x5FbDB2315678afecb367f032d93F642f64180aa3 "store(uint256)" 123 --rpc-url $RPC_URL --private-key $PRIVATE_KEY 
The command takes the address of the deployed contract, in double brackets goes the function signature followed by the value for the function, followed by the RPC_URL and the private key for our wallet.

Now, to read this value (getter function) we can use the `cast call` function:

    ❯ cast call 0x5FbDB2315678afecb367f032d93F642f64180aa3 "retrieve()"
    returns => ❯ 0x000000000000000000000000000000000000000000000000000000000000007b
This returns the hex value, so we can convert from hex to decimal:

    ❯ cast --to-base 0x000000000000000000000000000000000000000000000000000000000000007b dec






