.. index:: voting, ballot

.. _voting:

******
투표
******

다음 컨트랙트는 꽤 복잡하지만 Solidity가 가지고 있는 많은 특성들을 보여줍니다. 
이 컨트랙트는 투표 컨트랙트를 시행합니다. 물론, 전자 투표의 주요 문제점이라 한다면 
어떻게 올바르게 사람들에게 투표권을 줄 것인지에 대한 부분과 어떻게 조작을 막을 수 있는가일겁니다. 
여기서 모든 문제들을 해결하지는 않을 거지만, 최소한 투표가 어떻게 섬세하게 이루어지는지 
알아봄으로서 투표 결과가 **자동적이고 완전 무결하게** 나올 수 있도록 알아보도록 하겠습니다.

여기서 핵심은 무기명 투표의 각 옵션에 대한 짧은 이름을 제공함과 함께 투표 하나 당 하나의 컨트랙트를 생성하는 것입니다.
그럼 의장인 컨트랙트를 생성한 사람은 개인적으로 각 주소에 대해 투표 권리를 부여하게 됩니다.

이후 주소 뒤에 있는 사람들은 본인들 스스로 투표를 하거나 신뢰하는 타인에게 투표권을 부여합니다.

투표 시간 종료 시점에 ``winningProposal()`` 부분에서 최다 득표 수를 기록한 proposal를 반환합니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    /// @title Voting with delegation.
    contract Ballot {
        // 이 부분은 추후 변수들에 대해 사용될
        // 새로운 복잡한 타입을 선언합니다.
        // 이는 단일 투표자를 나타내고 있습니다.
        struct Voter {
            uint weight; // weight은 delegation에 의해 누적됩니다.
            bool voted;  // 만일 참일 경우, 해당 사람은 이미 투표했음을 가리킵니다.
            address delegate; // 투표권이 부여된 사람의 주소입니다.
            uint vote;   // 투표 proposal의 인덱스입니다.
        }

        // 이는 단일 proposal에 대한 타입입니다.
        struct Proposal {
            bytes32 name;   // 짧은 이름입니다 (최대 32바이트)
            uint voteCount; // 누적 득표수
        }

        address public chairperson;

        // 이 부분에선 각 유효한 주소에 대하여 `Voter` struct를 저장하는
        // state variable를 선언하고 있습니다.
        mapping(address => Voter) public voters;

        // 동적으로 크기가 결정되는 `Proposal` structs의 배열입니다.
        Proposal[] public proposals;

        /// `proposalNames`들 중 하나를 선택하는 새로운 무기명 투표를 생성합니다.
        constructor(bytes32[] memory proposalNames) {
            chairperson = msg.sender;
            voters[chairperson].weight = 1;

            // 제공된 각각의 proposal 이름에 대하여
            // 새로운 proposal 객체를 생성하고 
            // 이를 배열 끝 부분에 추가합니다.
            for (uint i = 0; i < proposalNames.length; i++) {
                // `Proposal({...})` 부분에서 임시적인 Proposal 객체를 생성하고 
                // `proposals.push(...)` 부분에서 `proposals` 끝 부분에 추가합니다.
                proposals.push(Proposal({
                    name: proposalNames[i],
                    voteCount: 0
                }));
            }
        }

        // `voter`에게 이 무기명 투표에 대한 투표권을 부여합니다.
        // `chairperson`에 의해서만 호출될 수 있습니다.
        function giveRightToVote(address voter) external {
            // 만일 `require`의 첫번째 인수가 `false`라고 평가된다면 
            // 실행이 종료되고
            // state에 적용된 모든 변경 사항과 Ether의 잔고가 다시 원상복귀됩니다.
            // 이전 EVM 버전에서는 가스를 모두 소비하곤 했으나, 
            // 현재는 더 이상 그러하지 않습니다.
            // 함수가 올바르게 호출되는지 체크하기 위하여 
            // `require`를 사용하는 것은 좋은 방법입니다.
            // 두번째 인수에 어떤 오류가 생겼는지 설명을 추가할 수도 있습니다.
            require(
                msg.sender == chairperson,
                "Only chairperson can give right to vote."
            );
            require(
                !voters[voter].voted,
                "The voter already voted."
            );
            require(voters[voter].weight == 0);
            voters[voter].weight = 1;
        }

        /// `to`를 가리키는 투표자에게 투표권을 위임합니다. 
        function delegate(address to) external {
            // 참조값을 할당합니다. 
            Voter storage sender = voters[msg.sender];
            require(!sender.voted, "You already voted.");

            require(to != msg.sender, "Self-delegation is disallowed.");

            // `to` 또한 위임되었다면 위임을 포워딩합니다. 
            // 보통 이러한 반복문은 굉장히 위험한데 
            // 너무 오랫동안 실행이 되었을 경우, 
            // 블록 안의 가스를 더 많이 필요로 할 수 있기 때문입니다. 
            // 이 경우, 위임은 실행되지 않지만,  
            // 다른 상황에서는 해당 반복 과정은 컨트랙트가 완전히 "막혀"버리는 현상이 생깁니다.
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

                // 위임 과정 내에서 허용되지 않은 반복을 찾습니다. 
                require(to != msg.sender, "Found loop in delegation.");
            }

            // `sender`가 참조값이므로
            // `voters[msg.sender].voted`를 수정합니다.
            sender.voted = true;
            sender.delegate = to;
            Voter storage delegate_ = voters[to];
            if (delegate_.voted) {
                // 만일 위임자가 이미 투표하였을 경우, 
                // 투표수에 직접적으로 추가합니다. 
                proposals[delegate_.vote].voteCount += sender.weight;
            } else {
                // 만일 위임자가 아직 투표하지 않았을 경우, 
                // weight에 추가합니다.
                delegate_.weight += sender.weight;
            }
        }

        /// `proposals[proposal].name` proposal에
        /// 여러분의 투표권 (여러분들께 위임된 투표권 포함)을 부여합니다.
        function vote(uint proposal) external {
            Voter storage sender = voters[msg.sender];
            require(sender.weight != 0, "Has no right to vote");
            require(!sender.voted, "Already voted.");
            sender.voted = true;
            sender.vote = proposal;

            // 만일 `proposal`이 배열의 범위를 벗어날 경우,
            // 이는 자동적으로 발생되며 모든 변경 사항들이 취소됩니다. 
            proposals[proposal].voteCount += sender.weight;
        }

        /// @dev 승리한 proposal를 
        /// 이전 투표수들을 합하여 계산합니다. 
        function winningProposal() public view
                returns (uint winningProposal_)
        {
            uint winningVoteCount = 0;
            for (uint p = 0; p < proposals.length; p++) {
                if (proposals[p].voteCount > winningVoteCount) {
                    winningVoteCount = proposals[p].voteCount;
                    winningProposal_ = p;
                }
            }
        }

        // proposals 배열 안에 포함된 승리자의 index를 얻기 위해 
        // winningProposal() 함수를 호출하고
        // 승리자의 이름이 반환됩니다.
        function winnerName() external view
                returns (bytes32 winnerName_)
        {
            winnerName_ = proposals[winningProposal()].name;
        }
    }


가능한 개선 사항들
=====================

현재로썬 모든 참가자들에게 투표권을 부여하기 위해 많은 트랜잭션이 필요합니다.
더 나은 방법이 있을까요?
