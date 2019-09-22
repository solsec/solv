Trust Fund 
==

```solidity
function withdraw() external ctf{
    require(allowancePerYear > 0, "No Allowances Allowed");
    checkIfYearHasPassed();    
    require(!withdrewThisYear, "Already Withdrew This Year");
    //allowancePerYear만큼 msg.sender에게 이더를 보내고 남은 가스를 넘겨줌
    //이때 가스가 부족하거나 잔고가 없어도 정상 종료될수 있음
    //여긴 잘 이해가 안됨.도움을 받아야겠음
    if (msg.sender.call.value(allowancePerYear)()){ 
        withdrewThisYear = true;
        numberOfWithdrawls = numberOfWithdrawls.add(1);
    }
}
```
