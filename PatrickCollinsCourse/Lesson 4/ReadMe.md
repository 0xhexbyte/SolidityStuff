# Lesson 4 - FundMe

## Library
1. The lesson explains the use of libraries.
2. Essentially, libraries are just solidity code written for a specific purpose that can be imported and used for further development.
3. Libraries can't have state variables and can't receive ether.
4. A library is embedded in a contract if all library functions are internal. (In a library, all functions have to be declared as - internal)
5. If not so, library must be deployed and linked before the contract is deployed.

=> Importing a library and then typing something like this:

    using PriceConverter for uint256;

where `PriceConverter` is the library in the `.sol` file, this means that all `uint256` values can access the functions inside `PriceConverter` library.

=> Also:

    require(msg.value.getConversionRate() >= minimumUsd, "not enough eth");

in an example as such we defined the getConversionRate() function in a library as such:

    function getConversionRate(uint256 ethAmount) internal view returns(uint256){ 
     uint256 ethPrice = getPrice(); 
     uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1e18; 
     return ethAmountInUsd;
     }

what we need to understand is when we type "msg.value" over there, it gets passed as the first arguement of the function `getConversionRate` and since it is `uint256`, it can access that function.

Lets say there was a second param for our function such as :

    function getConversionRate(uint256 ethAmount, uint256 secondParam) internal view returns(uint256){}

Then in this case, the value passed in the brackets where it gets implemented will be the second value, for example:

    require(msg.value.getConversionRate(secondParam) >= minimumUsd, "not enough eth");

so the first value is picked from the value before the `dot` operator and second one gets picked from the value being passed as an arguement.

## SafeMath
(SafeMath library by OpenZeppelin)
* The library is used to implement checks on mathematical operations so that the variables used in those operations do not overflow or underflow.
* The `unchecked` keyword is used to bypass any such checks, the sole aim to do that is to make the contract gas efficient.
* We should be using `unchecked` only when we're sure that the mathematical operation will never lead to either of the conditions i.e. overflow / underflow.

** However, from solidity code above 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

Just some extra reading (cause it feels professional):
https://ethereum.stackexchange.com/questions/113221/what-is-the-purpose-of-unchecked-in-solidity

## For Loop
A For Loop is used to repeat through a specific code for a specified number of times. An example will be:

        for(/* starting index, ending index, step amount*/)
So now to put it in an example:

            for(uint256 funderIndex = 0; funderIndex < funders.length; funderIndex++){
            /* Anything that goes within this code block will continue to get executed until the condition in the for loop stays true.*/
            }

## Resetting an Array
    funders = new address[](0);
Earlier we used `new` keyword to initiate a contract, whereas now it is being used to set the `funders` array to a brand new `address array`.

Different ways to send ETH to a contract:
1. Trasnfer
2. Send
3. Call

        msg.sender.transfer(address(this).balance);
The only thing we need to do here is to change the address (typecast it) to payable so it can receive ether. So the new code line will be:

1. transfer():

        payable(msg.sender).transfer(address(this).balance);
In Solidity, we can only send money to payable addresses. 
The transfer function is capped at 2300 gas, if more gas is used then it throws an error.

2. send():

        payable(msg.sender).send(address(this).balance);
The send function is also capped at 2300 gas, if more gas is consumed then it returns a boolean. In this case, we wouldn't be using the above code, but:

        bool sendSuccess = payable(msg.sender).transfer(address(this).balance);
        require(sendSuccess, "Send failed");
This way, even if it fails we will be able to revert the tx. We should not here that the `transfer` function automatically fails upon receiving an error.

3. call():

        (bool callSuccess, bytes memory dataReturned) = payable(msg.sender).call{value: address(this).balance}("");
The call function is very powerful. It is a low-level command and can be used to call any function in all of Ethereum without an ABI.
* Call function returns 2 values, a boolean and a bytes object.
* The empty quotes at last is the placeholder for a function to call, if any and any data returned from that call gets stored in the dataReturned variable.
* Since the bytes object is an array, data needs to be returned to the memory.
* Also, since we are not calling any function we can leave the code as such:

        (bool callSuccess, ) = payable(msg.sender).call{value: address(this).balance}("");
        require(callSuccess, "call failed");


## Constructor

If we look carefully at the code piece above, anyone can call the `withdraw()` function and withdraw the funds, which should only be accessible to the owner of the contract, this is where we use a constructor.

A constructor is a keyword in Solidity and can be simply defined as:

        constructor() {
            owner = msg.sender;
        }
This constructor gets called in the exact same transaction which is used to deploy the contract. Now, we can simply put a condition such as below to make sure only the owner calls the `withdraw()` function:

        function withdraw() public {
        require(msg.sender == owner, "Must be owner");
        for(uint256 funderIndex = 0; funderIndex < funders.length; funderIndex++)
        {
            address funder = funders[funderIndex];
            addressToAmountFunded[funder] = 0;
        }
    }

## Modifiers

Let's say in the above example, we intend to set that permission to multiple functions so that will be lots of copy-pasting. For situations like this, we have `modifiers`.

Modifier allows us to declare a keyword whose definition can be defined by us and then it can be added to all the other functions:

        modifier onlyOwner() {
            require(msg.sender == Owner, "Sender is not owner");
            _;
        }
So, now the keyword `onlyOwner` can be added to the function declaration as follows:

        function withdraw() public onlyOwner {}
The `_;` in the modifier means to carry forward with the function after executing the code inside the modifier. If this underscore is above the `require` function then it will first continue on with the code first and then check the condition that follows.


## Advanced Solidity Immutable & Constants
If we have variables that get set one time in our contract and never get changed, we can use some tools to make them more gas efficient.

constant:
When we add a constant keyword, the variable does not take storage spot and gets easier to read and thus reducing gas consumed.

            uint256 public constant minimumUsd = 5e18;
* Usually we have a common naming convention for `constant` variables and that is all caps. For example, the above line can be re-written as:

            uint256 public constant MINIMUM_USD = 5e18;

Another variable we only set one time is our `owner` variable. Variables that are set one time and outside of the line where they are declared can be marked as `immutable`. A common way to denote these is to have a leading `i_` before the variable name. For example:

        address public immutable i_owner;
        constructor(){
        i_owner = msg.sender;
    }

The reason that these keywords save gas is because instead of storing them in the storage slots, we save them directly in the bytecode of the contract.

## Advanced Solidity Custom Error:
With Solidity 0.8 we can define custom errors and save gas with require statements.

        error NotOwner();
So the onlyOwner modifier becomes:

        modifier onlyOwner() {
        if(msg.sender != i_owner) {revert NotOwner();}
        _;
        }
We can use the `revert()` function whenever and wherever it is required to revert the contract.


## Advanced Solidity Receive & Callback
















 
