.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
추상 컨트랙트 (Abstract Contracts)
******************

컨트랙트는 적어도 한개 이상의 그들의 함수가 구현이 되지 않았을 경우 추상화에 대한 표기가 필요합니다.
컨트랙트는 모든 함수들이 구현이 되어 있더라도 추상화로 표기될 수 있습니다.

이것은 하단의 예시에서 보여주는 것처럼 ``abstract`` 키워드를 사용하여 완료될 수 있습니다.
``utterance()`` 함수가 정의되었지만 구현부가 제공되지 않았기 때문에 해당 컨트랙트를 추상으로
정의해야 합니다. (구현 본문 ``{ }``이 제공되지 않았기 때문입니다)

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    abstract contract Feline {
        function utterance() public virtual returns (bytes32);
    }

이러한 추상 컨트랙트들은 직접적으로 인스턴스화될 수 없습니다. 모든 정의된 함수들에 대해 
추상 컨트랙트가 스스로 구현하고 있는 경우에도 마찬가지입니다. 추상 컨트랙트를 기본 클래스로 사용하는
방법은 다음 예제에 나와 있습니다:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    abstract contract Feline {
        function utterance() public pure virtual returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public pure override returns (bytes32) { return "miaow"; }
    }

컨트랙트가 추상 컨트랙트를 상속하고 재정의하여 구현되지 않은 모든 기능을 구현하지 않는 경우에도
추상으로 표기해야 합니다.

구현이 없는 함수는 구문이 매우 유사해 보이지만 :ref:`Function Type <function_types>`와 다릅니다.

다음은 구현이 없는 함수에 대한 예시입니다 (함수의 선언부):

.. code-block:: solidity

    function foo(address) external returns (address);

함수 타입을 가진 변수에 대한 선언의 예시입니다:

.. code-block:: solidity

    function(address) external returns (address) foo;

추상 컨트랙트는 컨트랙트의 정의를 구현에서 분리하여 더 나은 확장성과 자체 문서화를 제공하고
`Template method <https://en.wikipedia.org/wiki/Template_method_pattern>`_ 과 같은
패턴을 용이하게 하고 코드 중복을 제거합니다.
추상 컨트랙트는 인터페이스에서 메서드를 정의하는 것과 같은 방식으로 유용합니다. 추상 컨트랙트의
디자이너가 "만약 나의 자식 컨트랙트라면 이 메서드를 구현해야 한다" 라고 말하는 방식입니다.

.. note::

  추상 컨트랙트는 구현된 가상 함수를 구현되지 않은 것과 함께 오버라이드 할 수 없습니다.

