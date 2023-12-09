.. index:: ! error, revert

.. _errors:

*******************************
에러와 Revert 구문 (Errors and the Revert Statement)
*******************************

솔리디티의 에러들은 명령, 작업의 실패에 대한 이유를 설명하기에 가장 편리하고 가스 효율적인
방법을 제공합니다. 그것들은 내부적으로 그리고 컨트랙트의 외부적으로 정의될 수 있습니다.
(인터페이스들과 라이브러리들을 포함하여)

이것들은 현재 호출의 모든 변경 사항을 되돌리고 오류 데이터를 호출자에게 다시 전달하는
:ref:`revert statement <revert-statement>`와 함께 사용되어야 합니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    /// Insufficient balance for transfer. Needed `required` but only
    /// `available` available.
    /// @param available balance available.
    /// @param required requested amount to transfer.
    error InsufficientBalance(uint256 available, uint256 required);

    contract TestToken {
        mapping(address => uint) balance;
        function transfer(address to, uint256 amount) public {
            if (amount > balance[msg.sender])
                revert InsufficientBalance({
                    available: balance[msg.sender],
                    required: amount
                });
            balance[msg.sender] -= amount;
            balance[to] += amount;
        }
        // ...
    }

에러들은 오버로드되거나 오버라이드될 수 없지만 상속될 수 있습니다.
범위가 구별이 되는 한 동일한 오류는 여러 위치에서 정의될 수 있습니다.
에러들의 인스턴스들은 오직 ``revert`` 구문을 이용해서만 생성될 수 있습니다.

오류는 off-chain 컴포넌트로 돌아가거나 :ref:`try/catch statement <try-catch>`에서 잡아내기 위해
되돌리기 작업으로 호출자에게 전달되는 데이터를 생성합니다.
오류는 외부 호출에서 발생할 때만 catch를 수행할 수 있으며 내부 호출에서 발생하거나 동일한 함수
내부에서 발생하는 revert의 경우 catch를 수행할 수 없습니다.

만약 어떠한 파라미터도 제공하지 않았을 경우, 에러는 오직 데이터의 4바이트만을 필요로 하며
당신은 체인에 저장되지 않은 에러의 원인을 추가로 설명하기 위해
위와 같이 :ref:`NatSpec <natspec>`을 사용할 수 있습니다.
이것은 이것을 동시에 매우 값 싸면서 편리한 에러 리포팅 기능으로 만들어줍니다.

조금 더 구체적으로, 에러 인스턴스는 같은 이름과 유형의 함수에 대한 함수 호출과 같은 
방식으로 ABI로 인코딩된 다음 ``revert`` opcode에서 반환 데이터로 사용됩니다.
이것이 의미하는 것은 데이터가 4바이트의 셀렉터 이후This means that the data consists of a 4-byte selector followed by :ref:`ABI-encoded<abi>` data.
The selector consists of the first four bytes of the keccak256-hash of the signature of the error type.

.. note::
    동일한 이름의 다른 에러 또는 호출자가 구별할 수 없는 다른 위치에 정의된 에러로 컨트랙트를
    되돌릴 수 있습니다. ABI와 같은 외부의 경우 에러의 이름만 관련이 있으며 에러가 정의된
    컨트랙트나 파일은 관련이 없습니다.

만약 ``error Error(string)``을 정의할 수 있다면 ``require(condition, "description");`` 구문은 
``if (!condition) revert Error("description")``과 동일할 것입니다.
그러나 ``Error``는 기본으로 제공되는 built-in 타입이며 사용자 정의 코드에서 정의할 수 없습니다.

마찬가지로 실패한 ``assert`` 또는 유사한 조건의 경우 built-in 타입인
``Panic(uint256)`` 형식의 에러로 되돌아갑니다.

.. note::
    에러 데이터는 에러를 나타내는 데에만 사용해야 하며 제어 흐름을 위한 수단으로 사용해서는
    안됩니다. 그 이유는 내부 호출의 revert 데이터가 기본적으로 외부 호출 체인을 통해
    다시 전파되기 때문입니다. 이것이 의미하는 것은 내부 호출이 호출한 컨트랙트에서 가져온 것처럼
    보이는 데이터를 "위조"하여 되돌릴 수 있음을 의미합니다.