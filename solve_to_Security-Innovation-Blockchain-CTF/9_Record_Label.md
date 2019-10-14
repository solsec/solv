Record Label
==

```solidity
pragma solidity 0.4.24;

import "../CtfFramework.sol";
import "../../node_modules/openzeppelin-solidity/contracts/math/SafeMath.sol";


contract Royalties{

    using SafeMath for uint256;

    address private collectionsContract;
    address private artist;

    address[] private receiver;
    mapping(address => uint256) private receiverToPercentOfProfit;
    uint256 private percentRemaining;

    uint256 public amountPaid;

    constructor(address _manager, address _artist) public
    {
        collectionsContract = msg.sender;
        artist=_artist;

        receiver.push(_manager);
        receiverToPercentOfProfit[_manager] = 80;
        percentRemaining = 100 - receiverToPercentOfProfit[_manager];
    }

    modifier isCollectionsContract() { 
        require(msg.sender == collectionsContract, "Unauthorized: Not Collections Contract");
        _;
    }

    modifier isArtist(){
        require(msg.sender == artist, "Unauthorized: Not Artist");
        _;
    }

    function addRoyaltyReceiver(address _receiver, uint256 _percent) external isArtist{
        require(_percent<percentRemaining, "Precent Requested Must Be Less Than Percent Remaining");
        receiver.push(_receiver);
        receiverToPercentOfProfit[_receiver] = _percent;
        percentRemaining = percentRemaining.sub(_percent);
    }

    function payoutRoyalties() public payable isCollectionsContract{
        for (uint256 i = 0; i< receiver.length; i++){
            address current = receiver[i];
            uint256 payout = msg.value.mul(receiverToPercentOfProfit[current]).div(100);
            amountPaid = amountPaid.add(payout);
            current.transfer(payout);
        }
        msg.sender.call.value(msg.value-amountPaid)(bytes4(keccak256("collectRemainingFunds()")));
    }

    function getLastPayoutAmountAndReset() external isCollectionsContract returns(uint256){
        uint256 ret = amountPaid;
        amountPaid = 0;
        return ret;
    }

    function () public payable isCollectionsContract{
        payoutRoyalties();
    }
}

contract Manager{
    address public owner;

    constructor(address _owner) public {
        owner = _owner;
    }

    function withdraw(uint256 _balance) public {
        owner.transfer(_balance);
    }

    function () public payable{
        // empty
    }
}

contract RecordLabel is CtfFramework{

    using SafeMath for uint256;

    uint256 public funds;
    address public royalties;

    constructor(address _ctfLauncher, address _player) public payable
        CtfFramework(_ctfLauncher, _player)
    {
        royalties = new Royalties(new Manager(_ctfLauncher), _player);
        funds = funds.add(msg.value);
    }
    
    function() external payable ctf{
        funds = funds.add(msg.value);
    }


    function withdrawFundsAndPayRoyalties(uint256 _withdrawAmount) external ctf{
        require(_withdrawAmount<=funds, "Insufficient Funds in Contract");
        funds = funds.sub(_withdrawAmount);
        royalties.call.value(_withdrawAmount)();
        uint256 royaltiesPaid = Royalties(royalties).getLastPayoutAmountAndReset();
        uint256 artistPayout = _withdrawAmount.sub(royaltiesPaid); 
        msg.sender.transfer(artistPayout);
    }

    function collectRemainingFunds() external payable{
        require(msg.sender == royalties, "Unauthorized: Not Royalties Contract");
    }

}
```

## 풀이 과정

Funds 의 돈을 빼내야 한다.

코드를 살펴 보면,

```solidity
function withdrawFundsAndPayRoyalties(uint256 _withdrawAmount) external ctf{
        require(_withdrawAmount<=funds, "Insufficient Funds in Contract");
        funds = funds.sub(_withdrawAmount);
        royalties.call.value(_withdrawAmount)();
        // royalties에서 value를 call 한다.
        uint256 royaltiesPaid = Royalties(royalties).getLastPayoutAmountAndReset();
        uint256 artistPayout = _withdrawAmount.sub(royaltiesPaid); 
        msg.sender.transfer(artistPayout);
    }
```
>  royalties에서 value를 call 한다. 

> payoutRoyalties 로 연결 실행.

</br>

```solidy
    function payoutRoyalties() public payable isCollectionsContract{
        for (uint256 i = 0; i< receiver.length; i++){
            address current = receiver[i];
            uint256 payout = msg.value.mul(receiverToPercentOfProfit[current]).div(100);
            amountPaid = amountPaid.add(payout);
            current.transfer(payout);
        }
        msg.sender.call.value(msg.value-amountPaid)(bytes4(keccak256("collectRemainingFunds()")));
    }
```
> payout 금액에 recieiveToPercentOfProfi (초기 설정값 80)으로 되어있어서 가져가지 다 가져가지는 못할 것으로 보인다.
</br>


```solidity
royalties = new Royalties(new Manager(_ctfLauncher), _player);
```
> 하지만 royalties 는 컨트랙스 생성시에 Royalties 함수로 _player를 넣는다.

> 그리고 royalties 주소는 공개되어 있다.
 
    address public royalties;
    
</br>

```solidity
contract Royalties{
...
    address private collectionsContract;
    address private artist;

...
    constructor(address _manager, address _artist) public
    {
        collectionsContract = msg.sender;
        artist=_artist;

        receiver.push(_manager);
        receiverToPercentOfProfit[_manager] = 80;
        percentRemaining = 100 - receiverToPercentOfProfit[_manager];
    }
```
> 생성시에 _manager와 _artist 가 입력된다.
</br>

그래서 결국 수신자가 배열을 공유하기 때문에 _receiver 값을 _manager 주소와 동일하게 보내면 receiverToPercentOfProfit 값을 무시할 수 있다.

</br>
