# Sparkn  - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. Unexpected Behavior for contests](#H-01)

- ## Low Risk Findings
    - ### [L-01. No Winner Address(0) Check](#L-01)
    - ### [L-02. Implementation Upgradability](#L-02)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: CodeFox Inc.

### Dates: Aug 21st, 2023 - Aug 29th, 2023

[See more contest details here](https://www.codehawks.com/contests/cllcnja1h0001lc08z7w0orxx)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 1
   - Medium: 0
   - Low: 2


# High Risk Findings

## <a id='H-01'></a>H-01. Unexpected Behavior for contests            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-08-sparkn/blob/0f139b2dc53905700dd29a01451b330f829653e9/src/ProxyFactory.sol#L127

https://github.com/Cyfrin/2023-08-sparkn/blob/0f139b2dc53905700dd29a01451b330f829653e9/src/ProxyFactory.sol#L152

https://github.com/Cyfrin/2023-08-sparkn/blob/0f139b2dc53905700dd29a01451b330f829653e9/src/ProxyFactory.sol#L179

https://github.com/Cyfrin/2023-08-sparkn/blob/0f139b2dc53905700dd29a01451b330f829653e9/src/ProxyFactory.sol#L205

## Summary
Unexpected behavior regarding `setContest` and `deployProxyAndDistribute`

## Vulnerability Details
There are 3 scenarios:

1. The sponsors can sent tokens to a expired contest since there is no reset/check. This will result in locking of funds or  the next scenario.
2. It is possible to run `deployProxyAndDistribute` more than once. It will pass all checks and since there is funds in the contract, it will double spend and send tokens to winner addresses.
3. New contests with same `salt` will not be initialized even though the old contest has expired.

## Impact
Possible double spending and locking of funds in contract

## Tools Used
Manual review

## Recommendations
The best solution I can think of is setting:<br/>
```saltToCloseTime[salt]=0``` <br/>
after each of the `deployProxyAndDistribute` functions.

1. Vulnerability #1 is solved. Use a check for sponsors while sending tokens to the contract that the `saltToCloseTime[salt]>0`.
2. Vulnerability #2 is solved. `deployProxyAndDistribute` cannot be run more than once since there is check for `!saltToCloseTime[salt]>0`
3. Vulnerability #3 is solved. New contests with same `salt` can be set
		


# Low Risk Findings

## <a id='L-01'></a>L-01. No Winner Address(0) Check            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-08-sparkn/blob/0f139b2dc53905700dd29a01451b330f829653e9/src/Distributor.sol#L116

## Summary
No Address(0) check for Winners in the array
## Vulnerability Details
`winners` array length is checked. However, the individual addresses are not checked for 0x00 address. This may cause loss of  funds by sending tokens to Address(0)

## Impact
Loss of Funds. Irrecoverable

## Tools Used
Manual Review

## Recommendations
Put a address(0) check inside the loop which is used to send tokens to the winner addresses. In `Distributor.sol` 
```
for (uint256 i; i < winnersLength; ) {
            if(winners[i]==address(0)){
                revert Distributor__NoZeroAddress()();
            }
         //remaining logic
        }
         
```
## <a id='L-02'></a>L-02. Implementation Upgradability            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-08-sparkn/blob/0f139b2dc53905700dd29a01451b330f829653e9/src/ProxyFactory.sol

## Summary
In `ProxyFactory.sol`, the implementation address is required to be passed in every function as a parameter. This may not be best practice as this is prone to mistakes/typos.
Moreover, there is no upgradability function in the `ProxyFactory.sol`. In case there is any changes required to the implementation contract, the new address will have to be passed which again may result in mistakes.

## Vulnerability Details
The variable `implementation` which holds the address of the logic contract is required to be passed in as a parameter in each function. This is not required as the the address will be the same unless upgraded.

## Impact
If a contest is set with a wrong implementation address, there is no way to recover from that other than redeploying with the correct address again. 
Instead of manually passing the variable each time, it can be read from a state variable directly.

## Tools Used
Manual review

## Recommendations

Two steps:
1. Create a state variable called `s_implementation` and initialize it later using the next step
```
address public s_implementation;
```

2. Create an `upgrade` function which allows the OWNER ONLY to change the implementaion address
```
function upgradeImplementation(address _newImplementation) public onlyOwner{
if(_newImplementation==address(0)){
     revert ProxyFactory__NoZeroAddress();
}
     s_implementation=_newImplementation;
}
```


