Trust Fund 
==

```solidity
function withdraw() external ctf{
    require(allowancePerYear > 0, "No Allowances Allowed");
    checkIfYearHasPassed();
    require(!withdrewThisYear, "Already Withdrew This Year");
    if (msg.sender.call.value(allowancePerYear)()){
        withdrewThisYear = true;
        numberOfWithdrawls = numberOfWithdrawls.add(1);
    }
}
```
