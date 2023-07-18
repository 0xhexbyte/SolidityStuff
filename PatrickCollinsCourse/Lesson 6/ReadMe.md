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

For any blockchain, the RPC URL is the actual HTTPs endpoint that we interact and send our API calls to. Whenever we deploy a contract, send a transaction or interact with metamask, we are essentially making an API call to that RPC endpoint.













