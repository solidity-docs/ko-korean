.. index:: ! constant

.. _constants:

**************************************
Constant and Immutable State Variables
**************************************

상태 변수들은 ``constant``나 ``immutable``이라고 선언될 수 있습니다.
모든 케이스에서, 변수들은 컨트랙트가 생성된 이후에 수정될 수 없습니다.
``constant`` 변수들의 경우, 해당 값은 컴파일 타임에 고정되어야 하는 반면, ``immutable``의 경우에는
생성 시 여전히 할당될 수 있습니다.

또한 파일 레벨에서 ``constant`` 변수를 정의하는 것 또한 가능합니다.

컴파일러는 이러한 변수들을 위한 스토리지 슬롯을 예약하지 않으며, 그리고 모든 항목이 해당 값으로
대체됩니다.

일반적인 상태 변수들과 비교한다면, 컨트랙트의 가스에 대한 비용은 불변의 변수가 조금 더 
저렴합니다. 상수 변수의 경우, 할당된 표현식은 액세스되는 모든 위치에 복사되며 매번 재평가됩니다.
이는 로컬 최적화를 가능하게 합니다. 불변의 변수는 생성 시간에 한 번만 평가되며 해당 값은 액세스 되는
코드의 모든 위치에 복사됩니다. 이러한 값의 경우 32바이트가 더 적은 바이트에 맞더라도
예약됩니다. 이 때문에 상수 값은 때때로 불변 값보다 저렴할 수 있습니다.

모든 상수와 불변의 모든 유형이 지금 시점에 구현되는 것은 아닙니다. 지원되는 유일한 유형은
:ref:`strings <strings>` (오직 상수에만 해당) 그리고 :ref:`value types <value-types>`유형입니다.


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.4;

    uint constant X = 32**22 + 8;

    contract C {
        string constant TEXT = "abc";
        bytes32 constant MY_HASH = keccak256("abc");
        uint immutable decimals;
        uint immutable maxBalance;
        address immutable owner = msg.sender;

        constructor(uint _decimals, address _reference) {
            decimals = _decimals;
            // Assignments to immutables can even access the environment.
            maxBalance = _reference.balance;
        }

        function isBalanceTooHigh(address _other) public view returns (bool) {
            return _other.balance > maxBalance;
        }
    }


상수 (Constant)
========

``constant`` 변수의 경우, 값은 컴파일 타임에 상수화 되며, 변수가 선언된 위치에 할당해야 합니다.
스토리지에 액세스되거나 블록체인 데이터 (예를 들면 ``block.timestamp``, ``address(this).balance`` 또는
``block.number``)이거나 실행 데이터 (``msg.value`` 또는 ``gasleft()``)이거나 또는 외부 컨트랙트를
호출하게 만드는 모든 표현식은 허용되지 않습니다.
메모리 할당에 사이드 이펙트가 있을 수 있는 표현식은 허용되지만 다른 메모리 개체에 사이드 이펙트가 있을 수
있는 표현식은 허용되지 않습니다. 내장 함수인 ``keccak256``, ``sha256``, ``ripemd160``, ``ecrecover``,
``addmod`` 그리고 ``mulmod``는 모드 허용됩니다. (그러나 ``keccak256``을 제외하고는 모두 외부 계약을 호출합니다.)

메모리 할당자에 사이드 이펙트를 허용하는 이유는 룩업 테이블 (조회 테이블)과 같은 복잡한 객체를
구성할 수 있어야 하기 때문입니다.
이 기능은 아직 완전히 사용할 수 없습니다.

불변 (Immutable)
=========

``immutable``로 선언된 변수들은 ``constant``로 선언된 변수들보다 약간 덜 제한적입니다:
불변 변수들은 그들이 정의된 시점이나 컨트랙트의 생성자에서 임의의 값이 할당될 수 있습니다.
이는 한 번만 할당할 수 있으며 그 이후에는 생성 시간에도 읽을 수 있습니다.

컴파일러에 의해 생성된 컨트랙트의 생성 코드는 불변에 대한 모든 참조를 할당된 값으로
대체하여 반환되기 전에 컨트랙트의 런타임 코드를 수정합니다. 이것은 컴파일러가 생성한
런타임 코드를 블록체인에 실제로 저장된 런타임 코드와 비교할 때 중요합니다.

.. note::
  선언 시 할당된 불변 변수는 컨트랙트 생성자가 실행 중인 경우에만 초기화된 것으로
  간주합니다. 이것이 의미하는 것은 다른 불변 변수에 의존하는 값으로 임의의 불변 변수를
  인라인으로 초기화할 수 없다는 것을 의미합니다. 그러나 이는 컨트랙트의 생성자 내에서
  수행할 수 있습니다.

  이것은 특히 상속과 관련하여 상태 변수 초기화 및 생성자 실행 순서에 대한
  여러 서로 다른 해석에 대한 보호 장치라고 볼 수 있습니다.