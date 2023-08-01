## Testing Introduction
Testing code is essential to being a blockchain engineer as well as an auditor.
In remix, whenever we type @chainlink it automatically reaches out to npm package repository in order to fetch these contracts. However, in foundry, we need to specify our exact needs, so while setting up `FundMe.sol` we need to download those dependencies directly from github.

So for example if we need a package such as : `@smartcontractkit/chainlink-brownie-contracts` then we can run a forge command using:
```
forge install smartcontractkit/chainlink-brownie-contracts@0.6.1 --no-commit
```
The version after the `@` sign specifies which version do we intend to install. Now in order to map the `@chainlink/contracts` to the installed library we will create a "remapping".
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

## Tests
Not writing tests is considered equal to not writing mature code and not having a good understanding of how to audit so writing tests and reading them is a crucial skill.
The first part of our test is the setUp() function:
```
contract FundMeTest is Test {
    function setUp() external {}

    function testDemo() public {}
}
```

When the tests are run using `forge test`, the gas used is shown for `testDemo()` function, however the `setUp()` function is run first and a good way to demonstrate that is the following test code:
```
contract FundMeTest is Test {
    uint256 number = 1;

    function setUp() external {
        number = 2;
    }

    function testDemo() public {
        assertEq(number, 2);
    }
```

## Debugging Tests

Sometimes our tests might fail, in these circumstances, we can use the `console` library defined in the `forge-std/src/Test.sol` too.
```
//SPDX-License-Idnetifier: MIT

pragma solidity ^0.8.18;

import {Test, console} from "../lib/forge-std/src/Test.sol";
import {FundMe} from "../src/FundMe.sol";

contract FundMeTest is Test {
    FundMe fundme;

    function setUp() external {
        fundme = new FundMe(0x694AA1769357215DE4FAC081bf1f309aDC325306);
    }

    function testMinDollarIsFive() public {
        assertEq(fundme.MINIMUM_USD(), 5e18);
    }

    function testOwnerIsMsgSender() public {
        console.log(i_owner);
        console.log(msg.sender);
        assertEq(fundme.getOwner(), address(this));
    }
}

```

## Advanced Deploy Scripts


