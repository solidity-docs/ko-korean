.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

***********************
Structure of a Contract
컨트랙트의 구조
***********************

Contracts in Solidity are similar to classes in object-oriented languages.
Each contract can contain declarations of :ref:`structure-state-variables`, :ref:`structure-functions`,
:ref:`structure-function-modifiers`, :ref:`structure-events`, :ref:`structure-errors`, :ref:`structure-struct-types` and :ref:`structure-enum-types`.
Furthermore, contracts can inherit from other contracts.

솔리디티의 컨트랙트는 객체 지향 언어의 클래스와 비슷합니다.
각 컨트랙트는 :ref:`structure-state-variables`, :ref:`structure-functions`,
:ref:`structure-function-modifiers`, :ref:`structure-events`, :ref:`structure-errors`,
:ref:`structure-struct-types`, 그리고 :ref:`structure-enum-types`의 정의를 포함합니다.
더 나아가, 컨트랙트는 다른 컨트랙트로부터 상속될 수 있습니다.


There are also special kinds of contracts called :ref:`libraries<libraries>` and :ref:`interfaces<interfaces>`.

:ref:`libraries<libraries>` 와 :ref:`interfaces<interfaces>`라고 불리는 특별한 종류의 컨트랙트들 또한 있습니다.

The section about :ref:`contracts<contracts>` contains more details than this section,
which serves to provide a quick overview.

:ref:`contracts<contracts>`에 관한 섹션은 이 섹션보다 더 자세한 내용을 포함하고 있어,
개요를 간단히 제공할 수 있습니다. 

.. _structure-state-variables:

State Variables
상태 변수
===============

State variables are variables whose values are permanently stored in contract
storage.
상태 변수는 변수값이 컨트랙트 저장소에 영원히 저장되는 변수입니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract SimpleStorage {
        uint storedData; // State variable
        // ...
    }

See the :ref:`types` section for valid state variable types and
:ref:`visibility-and-getters` for possible choices for
visibility.

올바른 상태 변수 타입은 :ref:`types`을 참고해주시고,
가시성(visibility)을 위해 가능한 선택은 :ref:`visibility-and-getters`을 참고하세요.

.. _structure-functions:

Functions
함수
=========

Functions are the executable units of code. Functions are usually
defined inside a contract, but they can also be defined outside of
contracts.

함수는 실행가능한 코드 단위입니다. 함수는 주로 컨트랙트 내부에 정의하지만,
컨트랙트 외부에 정의할 수도 있습니다. 

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.7.1 <0.9.0;

    contract SimpleAuction {
        function bid() public payable { // 함수
            // ...
        }
    }

    // Helper function defined outside of a contract
    // 컨트랙트 외부에 정의된 Helper 함수
    function helper(uint x) pure returns (uint) {
        return x * 2;
    }

:ref:`function-calls` can happen internally or externally
and have different levels of :ref:`visibility<visibility-and-getters>`
towards other contracts. :ref:`Functions<functions>` accept :ref:`parameters and return variables<function-parameters-return-variables>` to pass parameters
and values between them.


:ref:`function-calls` 는 내외부적으로 발생할 수 있고 다른 컨트랙트를 향해 다른 레벨의 :ref:`visibility<visibility-and-getters>`을 가질 수 있습니다.
:ref:`Functions<functions>`는 :ref:`parameters and return variables<function-parameters-return-variables>`을 그들 사이의 파라미터와 변수를 보내기 위해 받아들입니다. 
.. _structure-function-modifiers:

Function Modifiers
함수 변경자
==================

Function modifiers can be used to amend the semantics of functions in a declarative way
(see :ref:`modifiers` in the contracts section).
함수 변경자는 함수의 기능을 수정하기 위해 선언적인 방식으로 사용됩니다.
(컨트랙스 섹션의 :ref:`modifiers`를 참고하세요.)
Overloading, that is, having the same modifier name with different parameters,
is not possible.
오버로딩, 즉 파라미터가 다른 동일한 변경자 이름을 가지는 것은 허용되지 않습니다.

Like functions, modifiers can be :ref:`overridden <modifier-overriding>`.
함수와 같이, 변경자는 :ref:`overridden <modifier-overriding>`될 수 있습니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // Modifier
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public view onlySeller { // Modifier usage
            // ...
        }
    }

.. _structure-events:

Events
이벤트
======

Events are convenience interfaces with the EVM logging facilities.
이벤트는 EVM 로깅 기능을 사용하는 편리한 인터페이스입니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.21 <0.9.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // 이벤트

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // 이벤트 발생
        }
    }

See :ref:`events` in contracts section for information on how events are declared
and can be used from within a dapp.
이벤트의 선언과 이벤트의 디앱 내에서의 사용 방법은 
컨트랙트 섹션의 :ref:`events`을 참고하세요.
.. _structure-errors:

Errors
에러
======

Errors allow you to define descriptive names and data for failure situations.
Errors can be used in :ref:`revert statements <revert-statement>`.
In comparison to string descriptions, errors are much cheaper and allow you
to encode additional data. You can use NatSpec to describe the error to
the user.

에러는 오류 상황에 대해서 서술명과 데이터를 정의할 수 있게 해줍니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity ^0.8.4;

    /// Not enough funds for transfer. Requested `requested`,
    /// but only `available` available.
    /// 송금할 자금이 부족합니다. 'requested'가 요청됩니다.
    /// 하지만 'available'만 사용가능합니다.

    error NotEnoughFunds(uint requested, uint available);

    contract Token {
        mapping(address => uint) balances;
        function transfer(address to, uint amount) public {
            uint balance = balances[msg.sender];
            if (balance < amount)
                revert NotEnoughFunds(amount, balance);
            balances[msg.sender] -= amount;
            balances[to] += amount;
            // ...
        }
    }

See :ref:`errors` in the contracts section for more information.
더 많은 정보는 컨트랙트 섹션의 :ref:`errors`를 참고하세요.
.. _structure-struct-types:

Struct Types
구조체 타입
=============

Structs are custom defined types that can group several variables (see
:ref:`structs` in types section).
구조체는 여러개의 변수들을 묶어서 정의할 수 있는 사용자 정의 타입입니다
(타입 섹션의 :ref:`structs` 를 참고하세요).

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract Ballot {
        struct Voter { // 구조체
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

Enum Types
열거형 타입
==========

Enums can be used to create custom types with a finite set of 'constant values' (see
:ref:`enums` in types section).
열거형은 한정된 '상수' 집합으로 구성된 사용자 정의 타입을 만들 때 사용됩니다.
(타입 섹션의 :ref:`enums`을 참고하세요. ).

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // 열거형
    }
