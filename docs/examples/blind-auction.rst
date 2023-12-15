.. index:: auction;blind, auction;open, blind auction, open auction

*************
완전 경쟁 입찰
*************

이번 섹션에서는 이더리움에서 완전 경쟁 입찰 컨트랙트를 생성하는 것이 얼마나 쉬운지 알아보도록 하겠습니다. 
이를 위해, 먼저 모든 사람이 입찰가를 볼 수 있는 공개 입찰을 시작한 후, 이 컨트랙트를 입찰 기간이 끝날 때까진 실제 입찰가를 볼 수 없는 비공개 입찰로 연장시켜 보겠습니다.

.. _simple_auction:

간단한 공개 입찰
===================

아래 간단한 공개 입찰 컨트랙트의 핵심은 입찰 기간 동안엔 누구든지 입찰가를 제시할 수 있다는 점입니다. 
입찰에는 입찰자가 본인이 제시한 입찰가에 종속하게끔 하도록 돈이나 Ether를 보내는 것을 이미 포함하고 있습니다. 
만일 제일 높은 입찰가가 제시될 경우, 직전의 제일 높은 가격으로 부른 입찰자에게 돈이 들어갑니다. 
입찰 기간이 종료된 이후 수혜자가 돈을 받기 위해선 컨트랙트는 수동적으로 호출되어야 하며 컨트랙트 스스로 활성화될 수 없습니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract SimpleAuction {
        // 입찰의 파라미터를 의미합니다. 
        // 여기서의 시간은 절대적 unix 타임스탬프 (1970-01-01 이후부터 초 단위)이거나
        // 초 단위의 타임을 의미합니다. 
        address payable public beneficiary;
        uint public auctionEndTime;

        // 입찰의 현재 상황을 가리킵니다. 
        address public highestBidder;
        uint public highestBid;

        // 이전 입찰에서 허용된 철회를 의미합니다.
        mapping(address => uint) pendingReturns;

        // 맨 마지막에 true로 설정함으로서 어떠한 변동도 허용하지 않습니다.
        // 기본적으로 `false` 값으로 시작됩니다. 
        bool ended;

        // 변동 사항으로 인해 송출되는 이벤트를 의미합니다.
        event HighestBidIncreased(address bidder, uint amount);
        event AuctionEnded(address winner, uint amount);

        // 실패 내역을 설명하는 에러들입니다.

        // 세 개 연속의 슬래쉬 코멘트를 natspec 코멘트라 합니다.
        // 유저에게 트랜잭션을 확정하라고 요구하거나 
        // 에러 나타날 때 보여집니다. 

        /// 입찰이 이미 종료되었습니다. 
        error AuctionAlreadyEnded();
        /// 이미 같거나 보다 높은 입찰가가 존재합니다.
        error BidNotHighEnough(uint highestBid);
        /// 입찰이 아직 끝나지 않았습니다. 
        error AuctionNotYetEnded();
        /// auctionEnd 함수가 이미 호출되었습니다. 
        error AuctionEndAlreadyCalled();

        /// 수혜자의 주소인 `beneficiaryAddress`을 대신하여 
        /// 입찰 시간인 `biddingTime`초를 넣어 
        /// 간단한 입찰을 생성합니다. 
        constructor(
            uint biddingTime,
            address payable beneficiaryAddress
        ) {
            beneficiary = beneficiaryAddress;
            auctionEndTime = block.timestamp + biddingTime;
        }

        /// 해당 트랜잭션과 함께 제시된 경매의 입찰가입니다. 
        /// 입찰가는 오직 경매에서 이기지 못했을 때에만 환불받을 수 있습니다. 
        function bid() external payable {
            // 여기선 어떠한 인수도 필요하지 않습니다.
            // 모든 정보는 이미 트랜잭션의 일부이기 때문입니다. 
            // Ether를 받기 위해서 함수에 payable이라는 키워드가 필요합니다. 

            // 만일 입찰 기간이 끝났다면 해당 호출을 모두 취소합니다. 
            if (block.timestamp > auctionEndTime)
                revert AuctionAlreadyEnded();

            // 만일 입찰가가 높지 않다면, 돈을 다시 보내줍니다.
            // (취소에 대한 명령문은 여태까지 받았던 돈들을 포함하여
            // 해당 함수에서 실행하는 모든 변동 사항들을 무효화합니다.)
            if (msg.value <= highestBid)
                revert BidNotHighEnough(highestBid);

            if (highestBid != 0) {
                // highestBidder.send(highestBid)를 이용하여 단순히 
                // 돈을 환불시키는 것에는 보안상 위험이 있습니다.  
                // 왜냐하면 신뢰할 수 없는 컨트랙트를 실행시킬 수도 있기 때문입니다.
                // 그렇기 때문에 항상 수혜자들이 본인의 돈을 직접
                // 인출하게끔 하는 것이 안전합니다. 
                pendingReturns[highestBidder] += highestBid;
            }
            highestBidder = msg.sender;
            highestBid = msg.value;
            emit HighestBidIncreased(msg.sender, msg.value);
        }

        /// 초과 입찰한 입찰가에 대하여 인출합니다.
        function withdraw() external returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 이 부분을 항상 0으로 설정하는 것이 중요합니다. 
                // 왜냐하면 `send`가 반환하기 전에 수혜자가 반환 호출의 한 부분으로써
                // 함수를 다시 호출할 수도 있기 때문입니다. 
                pendingReturns[msg.sender] = 0;

                // msg.sender는 `address payable` 타입이 아니며 and must be
                // `send()` 함수의 멤버를 사용하기 위해 를 사용함으로서
                // `payable(msg.sender)`를 사용하여 분명히 변환되었음을 알려야 합니다. 
                if (!payable(msg.sender).send(amount)) {
                    // No need to call throw here, just reset the amount owing
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// 입찰을 종료하고 수혜자에게 가장 높은 입찰가를 보냅니다.
        function auctionEnd() external {
            // 다음 세 가지 단계를 통해 
            // 다른 컨트랙트들과 상호작용하는 함수를 설계하는 것이 좋습니다
            // (예: 함수 호출 혹은 Ether를 보내는 등)
            // 1. 조건 확인하기 
            // 2. 액션 실행 (잠재적으로 조건을 바꿀 수도 있습니다)
            // 3. 다른 컨트랙트들과 상호작용하기 
            // 만일 이러한 단계들이 뒤죽박죽 섞이게 될 경우 
            // 다른 컨트랙트가 현재 컨트랙트를 다시 호출할 수 있고 
            // 이에 state를 수정하거나 (ether payout)과 같은 효과가 여러번 일어날 수 있습니다.
            // 만일 내부에서 호출된 함수들이 외부 컨트랙트와의 상호작용을 포함한다면
            // 이는 외부 컨트랙트들과 상호작용이 있었다고 고려되어야 합니다.

            // 1. 조건
            if (block.timestamp < auctionEndTime)
                revert AuctionNotYetEnded();
            if (ended)
                revert AuctionEndAlreadyCalled();

            // 2. 효과
            ended = true;
            emit AuctionEnded(highestBidder, highestBid);

            // 3. 상호작용
            beneficiary.transfer(highestBid);
        }
    }

블라인드 입찰
=============

The previous open auction is extended to a blind auction in the following. The
advantage of a blind auction is that there is no time pressure towards the end
of the bidding period. Creating a blind auction on a transparent computing
platform might sound like a contradiction, but cryptography comes to the
rescue.

위에 소개한 간단한 공개 입찰은 다음을 통해 블라인드 입찰이 됩니다. 블라인드 입찰의 장점은 입찰 기간 만료로 인한 시간적 압박이 없다는 것입니다.
투명한 컴퓨팅 플랫폼 위에서 블라인드 경매를 구축한다는 것이 모순적으로 들릴 수 있지만, 암호학이 이를 해결할 것입니다.

During the **bidding period**, a bidder does not actually send their bid, but
only a hashed version of it.  Since it is currently considered practically
impossible to find two (sufficiently long) values whose hash values are equal,
the bidder commits to the bid by that.  After the end of the bidding period,
the bidders have to reveal their bids: They send their values unencrypted and
the contract checks that the hash value is the same as the one provided during
the bidding period.

**입찰 기간** 동안, 입찰자는 자신의 입찰을 전송하지 않습니다. 대신 입찰의 해쉬된 버전을 전송할 뿐입니다.
해쉬 값이 같은 (충분히 긴) 두 변수 값을 찾는 것은 현재에 실질적으로 불가능하다고 여겨지기 때문에,  입찰자는 이러한 방식을 수용할 수 있습니다.
입찰 기간이 끝난 후, 입찰자들은 그들의 입찰을 공개해야 합니다: 그들은 암호화되지 않은 변수값을 보내고, 컨트랙트는 변수의 해쉬값이 입찰 기간동안 들어온 값과 같은지 확인합니다.     

Another challenge is how to make the auction **binding and blind** at the same
time: The only way to prevent the bidder from just not sending the money after
they won the auction is to make them send it together with the bid. Since value
transfers cannot be blinded in Ethereum, anyone can see the value.

또 다른 문제는 어떻게 입찰을 **가리는 것**과 동시에 **묶을 수 있는지**에 대한 것입니다. 경매에서 낙찰된 후 입찰자가 돈을 보내지 않는 것을 막는 유일한 방법은
입찰과 함꼐 돈을 보내도록 하는 것입니다.  이더리움 내에서는 변수 전송을 가릴 수 없고, 모두가 변수값을 볼 수 있기 때문입니다. 

The following contract solves this problem by accepting any value that is
larger than the highest bid. Since this can of course only be checked during
the reveal phase, some bids might be **invalid**, and this is on purpose (it
even provides an explicit flag to place invalid bids with high value
transfers): Bidders can confuse competition by placing several high or low
invalid bids.

다음의 컨트랙트는 가장 높은 입찰가보다 더 큰 모든 변수를 받아들임으로써 위의 문제를 해결합니다.
이는 물론 공개 단계에서만 확인이 가능하기 때문에, 일부 입찰은 무효가 될 수 있으며, 이는 고의적인 것입니다.
(높은 변수 값 이전으로 무효 입찰을 할수 있도록 명시적인 플래그를 제공하기도 합니다.): 입찰자들은 높거나 낮은 여러 무효 입찰 때문에 혼란을 겪을 수 있습니다.


.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract BlindAuction {
        struct Bid {
            bytes32 blindedBid;
            uint deposit;
        }

        address payable public beneficiary;
        uint public biddingEnd;
        uint public revealEnd;
        bool public ended;

        mapping(address => Bid[]) public bids;

        address public highestBidder;
        uint public highestBid;

        // Allowed withdrawals of previous bids
        // 이전 입찰의 철회가 허용됩니다.
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        // Errors that describe failures.
        // 실패 내역을 설명하는 에러들입니다.

        /// The function has been called too early.
        // 함수가 너무 일찍 호출되었습니다.
        /// Try again at `time`.
        // '제 시간'에 다시 시도해주세요.
        error TooEarly(uint time);
        /// The function has been called too late.
        // 함수가 너무 늦게 호출되었습니다.
        /// It cannot be called after `time`.
        // 'time' 이후에 호출될 수 없습니다.
        error TooLate(uint time);
        /// The function auctionEnd has already been called.
        // auctionEnd 함수가 이미 호출되었습니다.
        error AuctionEndAlreadyCalled();

        // Modifiers are a convenient way to validate inputs to
        // functions. `onlyBefore` is applied to `bid` below:
        // The new function body is the modifier's body where
        // `_` is replaced by the old function body.
        // Modifier는 함수에 대한 입력을 검증하는 편리한 방법입니다.
        // 'onlyBefore'는 아래의 'bid'에 적용됩니다:
        // 새로운 함수 body는 modifier의 body로, '_'가 기존의 함수 body로 대체됩니다.
        modifier onlyBefore(uint time) {
            if (block.timestamp >= time) revert TooLate(time);
            _;
        }
        modifier onlyAfter(uint time) {
            if (block.timestamp <= time) revert TooEarly(time);
            _;
        }

        constructor(
            uint biddingTime,
            uint revealTime,
            address payable beneficiaryAddress
        ) {
            beneficiary = beneficiaryAddress;
            biddingEnd = block.timestamp + biddingTime;
            revealEnd = biddingEnd + revealTime;
        }

        /// Place a blinded bid with `blindedBid` =
        /// keccak256(abi.encodePacked(value, fake, secret)).
        /// The sent ether is only refunded if the bid is correctly
        /// revealed in the revealing phase. The bid is valid if the
        /// ether sent together with the bid is at least "value" and
        /// "fake" is not true. Setting "fake" to true and sending
        /// not the exact amount are ways to hide the real bid but
        /// still make the required deposit. The same address can
        /// place multiple bids.
        /// 아래 함수를 통해 블라인드 입찰을 진행합니다.
        /// `blindedBid` = keccak256(abi.encodePacked(value, fake, secret)).
        /// 보낸 ether는 입찰이 공개단계에서 확실히 공개되었을 때만 환불됩니다.
        /// 입찰은 입찰과 함께 보낸 ether이 적어도 'value'이고
        /// 'fake'가 true가 아닐 때에만 유효합니다.
        /// 'fake'를 true로 설정하고 정확하지 않은 양을 이더를 보내는 것은
        /// 실제 입찰을 숨기는 방법이지만,  여전히 필요한 보증금을 지불합니다.
        /// 동일한 주소로 여러 번 입찰이 가능합니다.

        function bid(bytes32 blindedBid)
            external
            payable
            onlyBefore(biddingEnd)
        {
            bids[msg.sender].push(Bid({
                blindedBid: blindedBid,
                deposit: msg.value
            }));
        }

        /// Reveal your blinded bids. You will get a refund for all
        /// correctly blinded invalid bids and for all bids except for
        /// the totally highest.
        /// 블라인드된 입찰을 공개합니다. 
        /// 가장 높은 입찰을 제외한 무효가 확실한 모든 입찰들이 환불됩니다.
        function reveal(
            uint[] calldata values,
            bool[] calldata fakes,
            bytes32[] calldata secrets
        )
            external
            onlyAfter(biddingEnd)
            onlyBefore(revealEnd)
        {
            uint length = bids[msg.sender].length;
            require(values.length == length);
            require(fakes.length == length);
            require(secrets.length == length);

            uint refund;
            for (uint i = 0; i < length; i++) {
                Bid storage bidToCheck = bids[msg.sender][i];
                (uint value, bool fake, bytes32 secret) =
                        (values[i], fakes[i], secrets[i]);
                if (bidToCheck.blindedBid != keccak256(abi.encodePacked(value, fake, secret))) {
                    // Bid was not actually revealed.
                    // Do not refund deposit.
                    // 입찰은 확실히 공개되지 않습니다.
                    // 
                    continue;
                }
                refund += bidToCheck.deposit;
                if (!fake && bidToCheck.deposit >= value) {
                    if (placeBid(msg.sender, value))
                        refund -= value;
                }
                // Make it impossible for the sender to re-claim
                // the same deposit.
                bidToCheck.blindedBid = bytes32(0);
            }
            payable(msg.sender).transfer(refund);
        }

        /// 초과입찰된 입찰을 철회합니다.
        function withdraw() external {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // It is important to set this to zero because the recipient
                // can call this function again as part of the receiving call
                // before `transfer` returns (see the remark above about
                // conditions -> effects -> interaction).
                pendingReturns[msg.sender] = 0;

                payable(msg.sender).transfer(amount);
            }
        }

        /// 입찰을 종료하고, 수혜자에게 가장 큰 입찰을 보냅니다.
        function auctionEnd()
            external
            onlyAfter(revealEnd)
        {
            if (ended) revert AuctionEndAlreadyCalled();
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }

        // 이것은 계약 자체(또는 파생된 계약)으로부터만 호출이 가능한 "내적인" 기능입니다.
        function placeBid(address bidder, uint value) internal
                returns (bool success)
        {
            if (value <= highestBid) {
                return false;
            }
            if (highestBidder != address(0)) {
                // Refund the previously highest bidder.
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }
    }
