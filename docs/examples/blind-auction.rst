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

Blind Auction
=============

The previous open auction is extended to a blind auction in the following. The
advantage of a blind auction is that there is no time pressure towards the end
of the bidding period. Creating a blind auction on a transparent computing
platform might sound like a contradiction, but cryptography comes to the
rescue.

During the **bidding period**, a bidder does not actually send their bid, but
only a hashed version of it.  Since it is currently considered practically
impossible to find two (sufficiently long) values whose hash values are equal,
the bidder commits to the bid by that.  After the end of the bidding period,
the bidders have to reveal their bids: They send their values unencrypted and
the contract checks that the hash value is the same as the one provided during
the bidding period.

Another challenge is how to make the auction **binding and blind** at the same
time: The only way to prevent the bidder from just not sending the money after
they won the auction is to make them send it together with the bid. Since value
transfers cannot be blinded in Ethereum, anyone can see the value.

The following contract solves this problem by accepting any value that is
larger than the highest bid. Since this can of course only be checked during
the reveal phase, some bids might be **invalid**, and this is on purpose (it
even provides an explicit flag to place invalid bids with high value
transfers): Bidders can confuse competition by placing several high or low
invalid bids.


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
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        // Errors that describe failures.

        /// The function has been called too early.
        /// Try again at `time`.
        error TooEarly(uint time);
        /// The function has been called too late.
        /// It cannot be called after `time`.
        error TooLate(uint time);
        /// The function auctionEnd has already been called.
        error AuctionEndAlreadyCalled();

        // Modifiers are a convenient way to validate inputs to
        // functions. `onlyBefore` is applied to `bid` below:
        // The new function body is the modifier's body where
        // `_` is replaced by the old function body.
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

        /// Withdraw a bid that was overbid.
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

        /// End the auction and send the highest bid
        /// to the beneficiary.
        function auctionEnd()
            external
            onlyAfter(revealEnd)
        {
            if (ended) revert AuctionEndAlreadyCalled();
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }

        // This is an "internal" function which means that it
        // can only be called from the contract itself (or from
        // derived contracts).
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
