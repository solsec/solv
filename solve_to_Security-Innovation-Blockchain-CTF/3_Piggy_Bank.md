# Piggy_Bank

```solidity
pragma solidity 0.4.24;

import "../CtfFramework.sol";
import "../../node_modules/openzeppelin-solidity/contracts/math/SafeMath.sol";

contract PiggyBank is CtfFramework{

    using SafeMath for uint256;

    uint256 public piggyBalance;
    string public name;
    address public owner;

    constructor(address _ctfLauncher, address _player, string _name) public payable
        CtfFramework(_ctfLauncher, _player)
    {
        name=_name;
        owner=msg.sender;
        piggyBalance=piggyBalance.add(msg.value);
    }

    function() external payable ctf{
        piggyBalance=piggyBalance.add(msg.value);
    }


    modifier onlyOwner(){
        require(msg.sender == owner, "Unauthorized: Not Owner");
        _;
    }

    function withdraw(uint256 amount) internal{
        piggyBalance = piggyBalance.sub(amount);
        msg.sender.transfer(amount);
    }

    function collectFunds(uint256 amount) public onlyOwner ctf{
        require(amount<=piggyBalance, "Insufficient Funds in Contract");
        withdraw(amount);
    }
    //PiggyBank의 collectFunds 함수

}


contract CharliesPiggyBank is PiggyBank{

    uint256 public withdrawlCount;

    constructor(address _ctfLauncher, address _player) public payable
        PiggyBank(_ctfLauncher, _player, "Charlie")
    {
        withdrawlCount = 0;
    }

    function collectFunds(uint256 amount) public ctf{
        require(amount<=piggyBalance, "Insufficient Funds in Contract");
        withdrawlCount = withdrawlCount.add(1);
        withdraw(amount);
    }
    // PiggyBank에서 Derive된 CharliesPiggyBank의 collectFunds는 기존 PiggyBank의 collectFunds를 Override하였다. 따라서 function의 동작이 변경됨

}
```

## 풀이방법

```
 MyetherWallet에서 MetaMask를 연동한 후, 위 문제의 SmartContract 주소와 ABI를 입력한 후,
 collectFunds 함수를 호출하면 문제가 풀리게 된다.

 Override로 인해 원래 함수가 덮어써지게 되어 Owner가 아니라고 해당 함수를 호출할 수 있게 되었다.
```
