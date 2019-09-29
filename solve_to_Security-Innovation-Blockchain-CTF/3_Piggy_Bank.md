# Piggy_Bank

==

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

}
```
