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

A deploy script for our contract:
```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Script.sol";
import {FundMe} from "../src/FundMe.sol";

contract DeployFundMe is Script {
    function run() public {
        vm.startBroadcast();
        new FundMe(0x5FbDB2315678afecb367f032d93F642f64180aa3); //this is my locally deployed address)
        vm.stopBroadcast();
    }
}
```

## Forked Tests

When we run a `forge test` and don't give it a RPC URL, foundry spins up a local blockchain using anvil, attempts to run the tests and shuts it down after execution.
There are 4 different type of tests:
1. Unit - To test a specific part of the code
2. Integration - Testing how our code works with other parts of the code
3. Forked - Testing our code on a simulated real environment
4. Testing - Testing our code in a real environment that is not prod

There is another forge command to find out the amount of code that is being covered in our tests and that is:
```
forge coverage --rpc-url $SEPOLIA_RPC_URL
```
This command shows how much code is covered from all of our code files in our test cases.

## Refactoring: Testing Deploy Scripts
We can't have hardcoded addresses in our contract as that might create lot of extra work if when in future we need to change the address or for the matter, any hardcoded value.
FundMeTest.t.sol:
```
//SPDX-License-Idnetifier: MIT

pragma solidity ^0.8.18;

import {Test, console} from "../lib/forge-std/src/Test.sol";
import {FundMe} from "../src/FundMe.sol";
import {DeployFundMe} from "../script/DeployFundMe.s.sol";

contract FundMeTest is Test {
    FundMe fundMe;

    function setUp() external {
        // fundme = new FundMe(0x694AA1769357215DE4FAC081bf1f309aDC325306);
        DeployFundMe deployFundMe = new DeployFundMe();
        fundMe = deployFundMe.run();
    }

    function testMinDollarIsFive() public {
        assertEq(fundMe.MINIMUM_USD(), 5e18);
    }

    function testOwnerIsMessageSender() public {
        assertEq(fundMe.getOwner(), msg.sender);
    }

    function testPVVersion() public {
        uint256 version = fundMe.getVersion();
        assertEq(version, 4);
    }
}
```

DeployFundMe.s.sol:
```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Script} from "../lib/forge-std/src/Script.sol";
import {FundMe} from "../src/FundMe.sol";

contract DeployFundMe is Script {
    function run() external returns (FundMe) {
        vm.startBroadcast();
        FundMe fundMe = new FundMe(0x694AA1769357215DE4FAC081bf1f309aDC325306);
        vm.stopBroadcast();
        return fundMe;
    }
}

//forge script script/DeployFundMe.s.sol --rpc-url http://127.0.0.1:8545
```

When we use DeployFundMe to initiate the FundMe contract, the `vm.startBroadcast()` makes the funder = `msg.sender`.


## Refactoring Helper Config:
We still do pass the sepoliad price feed address, in order to replicate this locally we need to create a `Mock` contract. We can deploy a price feed locally on `anvil` and interact with it for our tests.
So, we make HelperConfig which helps us fetch the price of respective chains:
```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.18;
import {Script} from "../lib/forge-std/src/Script.sol";

contract HelperConfig {
    NetworkConfig public activeNetworkConfig;

    constructor() {
        if (block.chainid == 11155111) {
            activeNetworkConfig = getSepoliaEthConfig();
        } else if (block.chainid == 1) {
            activeNetworkConfig = getMainnetEthConfig();
        } else {
            activeNetworkConfig = getAnvilEthConfig();
        }
    }

    struct NetworkConfig {
        address priceFeed;
    }

    function getSepoliaEthConfig() public pure returns (NetworkConfig memory) {
        // price feed address\
        NetworkConfig memory sepoliaConfig = NetworkConfig({
            priceFeed: 0x694AA1769357215DE4FAC081bf1f309aDC325306
        });
        return sepoliaConfig;
    }

    function getAnvilEthConfig() public pure returns (NetworkConfig memory) {}

    function getMainnetEthConfig() public pure returns (NetworkConfig memory) {
        NetworkConfig memory ethConfig = NetworkConfig({
            priceFeed: 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419
        });
        return ethConfig;
    }
}

// 1. Deploy mocks when we are on a local chain
// 2. Keep track of contract addresses across different chains
// Sepolia ETH/USD
// Mainnet ETH/USD
```

This way if we intend to make any other getter function for a different chain, we can simply replicate the function and create another `else if` case in the `constructor()`.

## Refactoring Mocks
For mock, or anvil blockchain - those live contracts do not exist as they do on other chains which is where we need to deploy those contracts here ourselves. Since we deploy these contracts by ourselves for anvil, we will need to make the HelperConfig `is Script` so as to access the `vm` keyword.

In order to deploy our own price feed, we need a price feed contract as such:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * @title MockV3Aggregator
 * @notice Based on the FluxAggregator contract
 * @notice Use this contract when you need to test
 * other contract's ability to read data from an
 * aggregator contract, but how the aggregator got
 * its answer is unimportant
 */
contract MockV3Aggregator {
    uint256 public constant version = 4;

    uint8 public decimals;
    int256 public latestAnswer;
    uint256 public latestTimestamp;
    uint256 public latestRound;

    mapping(uint256 => int256) public getAnswer;
    mapping(uint256 => uint256) public getTimestamp;
    mapping(uint256 => uint256) private getStartedAt;

    constructor(uint8 _decimals, int256 _initialAnswer) {
        decimals = _decimals;
        updateAnswer(_initialAnswer);
    }

    function updateAnswer(int256 _answer) public {
        latestAnswer = _answer;
        latestTimestamp = block.timestamp;
        latestRound++;
        getAnswer[latestRound] = _answer;
        getTimestamp[latestRound] = block.timestamp;
        getStartedAt[latestRound] = block.timestamp;
    }

    function updateRoundData(
        uint80 _roundId,
        int256 _answer,
        uint256 _timestamp,
        uint256 _startedAt
    ) public {
        latestRound = _roundId;
        latestAnswer = _answer;
        latestTimestamp = _timestamp;
        getAnswer[latestRound] = _answer;
        getTimestamp[latestRound] = _timestamp;
        getStartedAt[latestRound] = _startedAt;
    }

    function getRoundData(
        uint80 _roundId
    )
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return (
            _roundId,
            getAnswer[_roundId],
            getStartedAt[_roundId],
            getTimestamp[_roundId],
            _roundId
        );
    }

    function latestRoundData()
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return (
            uint80(latestRound),
            getAnswer[latestRound],
            getStartedAt[latestRound],
            getTimestamp[latestRound],
            uint80(latestRound)
        );
    }

    function description() external pure returns (string memory) {
        return "v0.6/tests/MockV3Aggregator.sol";
    }
}
```

Integrating this with our HelperConfig:getAnvilEthConfig() function:
```
function getAnvilEthConfig() public returns (NetworkConfig memory) {
        // deploy the mock
        // return the mock
        vm.startBroadcast();
        MockV3Aggregator mockPriceFeed = new MockV3Aggregator(9, 2000e8);
        vm.stopBroadcast();

        NetworkConfig memory anvilConfig = NetworkConfig({
            priceFeed: address(mockPriceFeed)
        });
        return anvilConfig;
    }
```

## Magic Numbers
Instead of having random numbers harcoded in our code like we did above with the mockPriceFeed instance we should initialize them as constants and then use those constants in the functions to improve readability.
For example, rather than:
```
function getAnvilEthConfig() public returns (NetworkConfig memory) {
        // deploy the mock
        // return the mock
        vm.startBroadcast();
        MockV3Aggregator mockPriceFeed = new MockV3Aggregator(8,2000e8);
        vm.stopBroadcast();

        NetworkConfig memory anvilConfig = NetworkConfig({
            priceFeed: address(mockPriceFeed)
        });
        return anvilConfig;
    }
```
We can refactor it as:
```
uint8 public constant DECIMALS = 8;
int256 public constant INITIAL_PRICE = 2000e8;

function getAnvilEthConfig() public returns (NetworkConfig memory) {
        // deploy the mock
        // return the mock
        vm.startBroadcast();
        MockV3Aggregator mockPriceFeed = new MockV3Aggregator(
            DECIMALS,
            INITIAL_PRICE
        );
        vm.stopBroadcast();

        NetworkConfig memory anvilConfig = NetworkConfig({
            priceFeed: address(mockPriceFeed)
        });
        return anvilConfig;
    }
```
In the code above, if we already have an instance deployed then we wouldn't want to deploy another one for which, we will make use of 0 address and write the following check:
```
function getAnvilEthConfig() public returns (NetworkConfig memory) {
        if (activeNetworkConfig.priceFeed != address(0)) {
            return activeNetworkConfig;
        }
        vm.startBroadcast();
        MockV3Aggregator mockPriceFeed = new MockV3Aggregator(
            DECIMALS,
            INITIAL_PRICE
        );
        vm.stopBroadcast();

        NetworkConfig memory anvilConfig = NetworkConfig({
            priceFeed: address(mockPriceFeed)
        });
        return anvilConfig;
    }

```
Now if we already ran this function once then the address `priceFeed` won't be zero which will return a `true` boolean for our if confition and return the active running address. 

By practice we would like to default all variables to `private` as they are gas efficient.

The `prank()` cheatcode sets the msg.sender to a specific address for the next call. <br/>
Another cheatcode `makeAddr()`, creates an address derived from the provided `name` as a parameter.

The following is a good example of using `prank`:
```
 function testFundUpdatesFundedDataStructure() public {
        vm.prank(testAddr); //The next tx will be sent by user
        fundMe.fund{value: SEND_VALUE}();
        uint256 amountFunded = fundMe.getAddressToAmountFunded(testAddr); //now we removed the address(this) to testAddr because of prank()
        assertEq(amountFunded, SEND_VALUE);
    }
```
The above example will give us an error and the reason for that is because we did create a new user with `prank()` but the user does not have any funds. So we can use another cheatcode called:<br/>
`deal()` , allows us to set the balance for a new address.
```
 function setUp() external {
        DeployFundMe deployer = new DeployFundMe();
        (fundMe, helperConfig) = deployer.run();
        vm.deal(USER, STARTING_USER_BALANCE);
    }
```

However, hoax() allows us to do both of these at once:
```
function testWithDrawFromMultipleFunders() public funded {
        uint160 numberOfFunders = 10;
        uint160 startingFunderIndex = 2;
        for (
            uint160 i = startingFunderIndex;
            i < numberOfFunders + startingFunderIndex;
            i++
        ) {
            // we get hoax from stdcheats
            // prank + deal
            hoax(address(i), STARTING_USER_BALANCE);
            fundMe.fund{value: SEND_VALUE}();
        }

        uint256 startingFundMeBalance = address(fundMe).balance;
        uint256 startingOwnerBalance = fundMe.getOwner().balance;

        vm.startPrank(fundMe.getOwner());
        fundMe.withdraw();
        vm.stopPrank();

        assert(address(fundMe).balance == 0);
        assert(
            startingFundMeBalance + startingOwnerBalance ==
                fundMe.getOwner().balance
        );
        assert(
            (numberOfFunders + 1) * SEND_VALUE ==
                fundMe.getOwner().balance - startingOwnerBalance
        );
    }
```
## Chisel
Chisel provides us with an interactive terminal and allows us to run solidity command line by line.

## Storage
Storage works as a giant list associated with the contract where every single variable and every single value in the storage section is stored in a 32 byte slot, in the storage array. <br/>
Similarly, the next variable will go onto the second index position which is [1].

![image](https://github.com/Mrig26/SolidityStuff/assets/36241150/b61da961-1af9-485c-b4fe-5408fdf3c71d)


All these values get modified into their hex versions to get stored in the storage array.<br/>

For any variable or mapping which is dynamic, the elements inside the array are stored using some hashing function.
![image](https://github.com/Mrig26/SolidityStuff/assets/36241150/0b08dbcb-9733-43bf-a1cb-e97172087528)

Interestingly, `constant` and `immutable` variables do not take up storage slots and the reason is that they are a part of the bytecode. <br/>
Any variables inside a function call are only there for the duration of execution, they do not persist. So such variables do not get added to the storage array. They get added in their own memory data structure which gets deleted after the function execution.<br/>

Now coming to `memory`, we also refer as string memory and we do so because a string is a dynamically sized array. We need to tell solidity if we will store it in a storage location or memory location where we wipe it. Arrays and Mappings can take up a lot more space so solidity wants to know where are we working with this.

Another useful command is `forge inspect` to see the storage structure of the contract. 
