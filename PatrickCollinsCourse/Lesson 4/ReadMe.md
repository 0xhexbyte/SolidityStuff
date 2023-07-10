# Lesson 4

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
function getConversionRate(uint256 ethAmount) internal view returns(uint256){
        uint256 ethPrice = getPrice();
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1e18;
        return ethAmountInUsd;
    }
`
what we need to understand is when we type "msg.value" over there, it gets passed as the first arguement of the function `getConversionRate` and since it is `uint256`, it can access that function.

Lets say there was a second param for our function such as :

`function getConversionRate(uint256 ethAmount, uint256 secondParam) internal view returns(uint256){}`

Then in this case, the value passed in the brackets where it gets implemented will be the second value, for example:
`
        require(msg.value.getConversionRate(secondParam) >= minimumUsd, "not enough eth");
`
so the first value is picked from the value before the `dot` operator and second one gets picked from the value being passed as an arguement.
