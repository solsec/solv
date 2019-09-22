Lock Box
==
```solidity
pragma solidity 0.4.24;

import "../CtfFramework.sol";

contract Lockbox1 is CtfFramework{

    uint256 private pin;

    constructor(address _ctfLauncher, address _player) public payable
        CtfFramework(_ctfLauncher, _player)
    {
        pin = now%10000;
    }
    
    function unlock(uint256 _pin) external ctf{
        require(pin == _pin, "Incorrect PIN");
        msg.sender.transfer(address(this).balance);
    }

}
```    
## 결과