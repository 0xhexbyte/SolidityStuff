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













