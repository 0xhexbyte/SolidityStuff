# Lesson 4

## SafeMath
The lesson teaches us about SafeMath library by OpenZeppelin.
* The library is used to implement checks on mathematical operations so that the variables used in those operations do not overflow or underflow.
* The `unchecked` keyword is used to bypass any such checks, the sole aim to do that is to make the contract gas efficient.
* We should be using `unchecked` only when we're sure that the mathematical operation will never lead to either of the conditions i.e. overflow / underflow.

However, from solidity code above 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

Just some extra reading (cause it feels professional):
https://ethereum.stackexchange.com/questions/113221/what-is-the-purpose-of-unchecked-in-solidity

## For Loop
