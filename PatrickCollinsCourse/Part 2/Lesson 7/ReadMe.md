## Testing Introduction
Testing code is essential to being a blockchain engineer as well as an auditor.
In remix, whenever we type @chainlink it automatically reaches out to npm package repository in order to fetch these contracts. However, in foundry, we need to specify our exact needs, so while setting up `FundMe.sol` we need to download those dependencies directly from github.

So for example if we need a package such as : `@smartcontractkit/chainlink-brownie-contracts` then we can run a forge command using:
```
forge install smartcontractkit/chainlink-brownie-contracts@0.6.1 --no-commit
```
The version after the `@` sign specifies which version do we intend to install. Now in order to map the @chainlink/contracts to the installed library we will create a "remapping".
To do so, we will open the foundry.toml file and put an entry as such:
```
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
remappings = ["@chainlink/contracts/=lib/chainlink-brownie-contracts/contracts"]
```
* Whenever we name errors, we should follow naming the error starting with the contract name, then 2 underscores and followed by error name. For example:
```
error FundMe__NotOwner();
```
