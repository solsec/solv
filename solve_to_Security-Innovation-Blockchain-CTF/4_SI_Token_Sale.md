# SI Token Sale

```solidity
pragma solidity 0.4.24;

import "../CtfFramework.sol";

// https://github.com/OpenZeppelin/openzeppelin-solidity/blob/v1.8.0/contracts/token/ERC20/StandardToken.sol
import "../StandardToken.sol";

contract SIToken is StandardToken {

    using SafeMath for uint256;

    string public name = "SIToken";
    string public symbol = "SIT";
    uint public decimals = 18;
    uint public INITIAL_SUPPLY = 1000 * (10 ** decimals);

    constructor() public{
        totalSupply_ = INITIAL_SUPPLY;
        balances[this] = INITIAL_SUPPLY;
    }
}

contract SITokenSale is SIToken, CtfFramework {

    uint256 public feeAmount;
    uint256 public etherCollection;
    address public developer;

    constructor(address _ctfLauncher, address _player) public payable
        CtfFramework(_ctfLauncher, _player)
    {
        feeAmount = 10 szabo;
        //feeAmount는 10 szabo로 선언되어 있다.(값이 고정)
        developer = msg.sender;
        purchaseTokens(msg.value);
    }

    function purchaseTokens(uint256 _value) internal{
        require(_value > 0, "Cannot Purchase Zero Tokens");
        require(_value < balances[this], "Not Enough Tokens Available");
        balances[msg.sender] += _value - feeAmount;
        //이 부분에 취약점이 존재하는데, feeAmount가 고정값이므로 underFlow를 발생시킬 수 있다.
        balances[this] -= _value;
        balances[developer] += feeAmount;
        etherCollection += msg.value;
    }
    // 해당 함수는 internal로 선언되어 있다.

    function () payable external ctf{
        purchaseTokens(msg.value);
    }
    //하지만 fallback 함수가 해당 함수를 호출 할 수 있게 한다.

    // Allow users to refund their tokens for half price ;-)
    function refundTokens(uint256 _value) external ctf{
        require(_value>0, "Cannot Refund Zero Tokens");
        transfer(this, _value);
        etherCollection -= _value/2;
        msg.sender.transfer(_value/2);
    }

    function withdrawEther() external ctf{
        require(msg.sender == developer, "Unauthorized: Not Developer");
        require(balances[this] == 0, "Only Allowed Once Sale is Complete");
        msg.sender.transfer(etherCollection);
    }

}
```

## 풀이방법

```
해당 Contract 주소에 10 Sazabo보다 적은 값을 보내면 underFlow가 발생하면서 문제가 풀리게 된다.
```
