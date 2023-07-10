# Lesson 4 - FundMe

## Library
1. The lesson explains the use of libraries.
2. Essentially, libraries are just solidity code written for a specific purpose that can be imported and used for further development.
3. Libraries can't have state variables and can't receive ether.
4. A library is embedded in a contract if all library functions are internal. (In a library, all functions have to be declared as - internal)
5. If not so, library must be deployed and linked before the contract is deployed.

=> Importing a library and then typing something like this:
    `using PriceConverter for uint256;`
where `PriceConverter` is the library in the `.sol` file, this means that all `uint256` values can access the functions inside `PriceConverter` library.

=> Also:

  `require(msg.value.getConversionRate() >= minimumUsd, "not enough eth");`

in an example as such we defined the getConversionRate() function in a library as such:

`
function getConversionRate(uint256 ethAmount) internal view returns(uint256){` <br />
        `uint256 ethPrice = getPrice();` \n
        `uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1e18;`\n
        `return ethAmountInUsd;`\n
    `}`\n

what we need to understand is when we type "msg.value" over there, it gets passed as the first arguement of the function `getConversionRate` and since it is `uint256`, it can access that function.

Lets say there was a second param for our function such as :

`function getConversionRate(uint256 ethAmount, uint256 secondParam) internal view returns(uint256){}`

Then in this case, the value passed in the brackets where it gets implemented will be the second value, for example:
`
        require(msg.value.getConversionRate(secondParam) >= minimumUsd, "not enough eth");
`
so the first value is picked from the value before the `dot` operator and second one gets picked from the value being passed as an arguement.

## SafeMath
(SafeMath library by OpenZeppelin)
* The library is used to implement checks on mathematical operations so that the variables used in those operations do not overflow or underflow.
* The `unchecked` keyword is used to bypass any such checks, the sole aim to do that is to make the contract gas efficient.
* We should be using `unchecked` only when we're sure that the mathematical operation will never lead to either of the conditions i.e. overflow / underflow.

However, from solidity code above 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

Just some extra reading (cause it feels professional):
https://ethereum.stackexchange.com/questions/113221/what-is-the-purpose-of-unchecked-in-solidity

## For Loop
