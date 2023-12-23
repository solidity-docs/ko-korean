##################################
표현식과 제어 구조
##################################

.. index:: ! parameter, parameter;input, parameter;output, function parameter, parameter;function, return variable, variable;return, return


.. index:: if, else, while, do/while, for, break, continue, return, switch, goto

제어 구조
===================

대부분의 중괄호 언어(해석 : 중괄호를 사용하는 언어. ex. C, JavaScript) 제어 구조는 솔리디티에서 사용가능합니다.

C언어나 JavaScript로부터 알려진 ``if``, ``else``, ``while``, ``do``, ``for``, ``break``, ``continue``, ``return`` 등의 일반적인 코드 등이 있습니다.

솔리디티는 또한  ``try`` / ``catch`` 구문의 형태로 예외 처리를 지원합니다만, 
:ref:`external function calls <external-function-calls>` 과 컨트랙트 생성 호출을 위해서만 사용합니다.
에러는 :ref:`revert statement <revert-statement>`를 사용하면서 생성될 수 있습니다.

괄호는 조건문에서 생략될 수 *없지만*, 중괄호는 한줄로 된 코드문에 대해서는 생략할 수 있습니다.

솔리디티에는 C와 JavaScript처럼 불리언(boolean)자료형이 아닌 타입을 불리언 타입으로 바꾸는 기능이 없다는 걸 명심하세요.
따라서  ``if (1) { ... }`` 는 솔리디티에서 *사용할 수 없는* 표현입니다.

.. index:: ! function;call, function;internal, function;external

.. _function-calls:

함수 호출
==============

.. _internal-function-calls:

내부 함수 호출
-----------------------

현재 컨트랙트의 함수들은 아래 말도 안되는 예제에서 볼 수 있듯이 직접적으로 ("내부적으로"), 또는 재귀적으로 부를 수 있습니다.

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    // 아래 코드는 경고를 보고합니다
    contract C {
        function g(uint a) public pure returns (uint ret) { return a + f(); }
        function f() internal pure returns (uint ret) { return g(7) + f(); }
    }

이러한 함수 호출들은 EVM 내부의 간단한 점프로 번역됩니다. 
이는 현재 메모리가 지워지지 않는 효과를 가집니다. 즉 메모리 참조를 내부적으로 호출하는 
함수에 전달하는 것은 매우 효과적입니다. 오직 같은 컨트랙트 인스턴트의 함수들만 내부적으로
호출될 수 있습니다.

모든 내부 함수 호출이 적어도 하나의 스택 슬롯을 사용하고 1024개의 슬롯만을 사용
할 수 있기 때문에, 과도한 재귀를 피해야 합니다

.. _external-function-calls:

외부 함수 호출
-----------------------

함수는 ``this.g(8);`` 와 ``c.g(2);`` 등의 표기법을 사용하여 부를 수도 있는데, 여기서
``c`` 는 컨트랙트 인스턴스이고 ``g``는 ``c``에 속한 함수입니다. 
두 방법 중 어느쪽이든 ``g`` 를 호출하는 것은 점프를 직접 사용하지 않고 메세지 호출을 
사용하여 외부적으로 호출하게 되는 결과를 낳습니다.
``this`` 위의 함수 호출은 아직 실질적인 컨트랙트가 생성되지 않았기 때문에 
생성자에서 사용할 수 없습니다.

다른 컨트랙트들의 함수들은 외부적으로 호출되어야 합니다. 외부 호출에 대해,
모든 함수의 매개변수들은 메모리로 복사되어야 합니다.

.. note::

    컨트랙트에서 다른 컨트랙트로의 함수 호출은 자체적인 트랜잭션을 생성하지 않습니다.
    이는 전체적인 트랜잭션의 한 부분으로써의 메세지 호출입니다.


다른 컨트랙트의 함수를 호출할 때, 호출과 함께 보내지는 웨이(Wei) 또는 가스(gas)를  ``{value: 10, gas: 10000}``
특별한 옵션을 통해 특정할 수 있습니다.
가스비을 명시적으로 지정하는 것은 권장되지 않는데, 명령코드의 가스비가 미래에 바뀔 수도 있기 때문입니다.
컨트랙트에 보내는 어떤 웨이든 그 컨트랙트의 총 잔고에 추가됩니다. 

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.6.2 <0.9.0;

    contract InfoFeed {
        function info() public payable returns (uint ret) { return 42; }
    }

    contract Consumer {
        InfoFeed feed;
        function setFeed(InfoFeed addr) public { feed = addr; }
        function callFeed() public { feed.info{value: 10, gas: 800}(); }
    }


``info`` 함수와 함께 ``payable``변경자를 사용할 필요가 있는데, 사용하지 않는다면 
``value``옵션은 이용가능하지 않게 되기 때문입니다.

.. 경고::

  ``feed.info{value: 10, gas: 800}``이 함수 호출과 함께 전송된 ``gas``비와 양을 로컬로
  설정하고 끝에 있는 괄호가 실제 호출을 수행할 수 있도록 주의해야 합니다.
  따라서 ``feed.info{value: 10, gas: 800}``은 함수를 호출하지 않고 ``value``와 ``gas`` 세팅이
  손실되어, ``feed.info{value: 10, gas: 800}()`` 만 함수 호출을 수행합니다.


EVM이 항상 성공하기 위해 존재하지 않는 컨트랙트를 호출하는 것을 고려하는 사실 때문에,
솔리디티는 호출되려고 하는 컨트랙트가 확실히 존재하는지(코드를 포함하는지) 확인하고
그렇지 않으면 예외를 발생시키기 위해 ``extcodesize``명령 코드를 사용합니다.

컨트랙트 인스턴스보다 주소에서 작동하는 :ref:`low-level calls <address_related>`의 경우 
이러한 확인이 수행되지 않는다는 것을 명심하십시오.

.. note::

    :ref:`precompiled contracts <precompiledContracts>`에 높은 수준의 호출을 사용할 때는
    코드를 실행하고 데이터를 반환할 수 있다고 해도 컴파일러가 위 논리에 따라 호출이 존재하지 
    않는다고 여기기 때문에 주의하십시오.
    
.

함수 호출은 호출된 컨트랙트 자체에서 예외를 발생시키거나 가스가 다 떨어졌을 경우
예외를 발생시키기도 합니다.

.. 경고::

    다른 컨트랙트의 상호작용은 잠재적인 위험을 초래하는데, 특히 컨트랙의 소스코드를 미리 알수 없는 경우에
    그렇습니다. 현재 컨트랙트는 호출된 컨트랙트의 통제권을 양도하고 잠재적으로 모든 것을 수행할 수 있습니다.
    심지어 호출된 컨트랙트가 알려진 상위 컨트랙트로부터 상속받더라도, 상속 계약은 올바른 인터페이스만
    갖추면 됩니다. 하지만 컨트랙트의 구현은 완전히 자의적일 수 있으며, 위험을 초래할 수 있습니다.
    또한 시스템의 다른 컨트랙트를 호출하거나 첫 번째 호출이 리턴되기 전에 다시 호출 컨트랙트로 
    돌아오는 경우를 대비해야 합니다. 이는
    호출된 컨트랙트는 함수를 통해 호출하는 컨트랙트의 상태 변수를 바꿀 수 있다는 것을 의미합니다.
    예를 들어, 컨트랙트의 상태 변수를 변경하여 컨트랙트가 재진입 악용에 취약하지 않게 한 후
    외부 함수에 대한 호출이 발생하는 방식으로 함수를 작성하십시오. 

.. note::

    솔리디티 0.6.2 이전에는 변수값과 가스를 특정하는 추천되는 방식은 ``f.value(x).gas(g)()``
    을 사용하는 것이었습니다. 이는 솔리디티 0.6.2에서 비난받았고 솔리디티 0.7.0 이후로 더 이상
    사용되지 않습니다.


지정 호출과 익명 함수 매개변수
---------------------------------------------

함수 호출 인수는 다음과 같이 ``{ }`` 로 둘러싸인 경우 순서가 어떻든 이름으로 지정할 수 있습니다. 

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract C {
        mapping(uint => uint) data;

        function f() public {
            set({value: 2, key: 3});
        }

        function set(uint key, uint value) public {
            data[key] = value;
        }

    }

함수 매개변수명 생략
--------------------------------


사용하지 않은 매개변수(특히 리턴 매개변수)의 이름은 생략할 수 있습니다.
이러한 매개변수는 여전히 스택에 존재할 것이지만, 접근할 수 없습니다. 

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    contract C {
        // omitted name for parameter
        function func(uint k, uint) public pure returns(uint) {
            return k;
        }
    }


.. index:: ! new, contracts;creating

.. _creating-contracts:

``new``를 통해 컨트랙트 생성하기
==============================

컨트랙트는 다른 컨트랙트를 ``new``키워드를 통해 생성할 수 있습니다. 만들어진 컨트랙트의
전체 코드는 생성하는 컨트랙트가 컴파일될 때 알려져야 하므로 재귀적 생성-의존성은 불가능합니다.

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract D {
        uint public x;
        constructor(uint a) payable {
            x = a;
        }
    }

    contract C {
        D d = new D(4); // C의 생성자처럼 실행됩니다.

        function createD(uint arg) public {
            D newD = new D(arg);
            newD.x();
        }

        function createAndEndowD(uint arg, uint amount) public payable {
            // 생성과 함께 이더 보내기
            D newD = new D{value: amount}(arg);
            newD.x();
        }
    }

위 예제에서 보았듯이, ``value``옵션을 사용하여  ``D``를 생성함과 동시에 이더를 보내는 것이
가능합니다. 하지만 가스의 양을 제한하는 것은 불가능합니다. 만약 (out of stack, not enough balance 
또는 다른 문제들로 인해) 생성에 실패하면 예외가 발생합니다.

솔트 컨트랙트 생성 / create2
-----------------------------------

컨트랙트를 생성할 때, 컨트랙트의 주소는 생성하는 컨트랙트의 주소와 각 컨트랙트 생성과 함께
증가하는 카운터로부터 계산됩니다.

만약 (bytes32 값의) ``salt``옵션을 명시한다면, 컨트랙트 생성은 새로운 계약 주소를 마련하기 위해
다른 매커니즘을 사용할 것입니다.

이는 생성하는 컨트랙트의 주소와, 주어진 salt값, 생성된 컨트랙트의 바이트코드, 그리고
생성자 인수로부터 주소를 계산할 것입니다.

특히, 카운터 ("nonce")는 사용되지 않습니다. 이는 컨트랙트를 생성할 때 유연성을 더해 줍니다:
새로운 컨트랙트가 생성되기 전에 컨트랙트의 주소를 도출할 수 있습니다.
더 나아가, 생성하는 컨트랙트가 잠시 다른 컨트랙트를 생성하는 동안에도 이 주소를 사용할 수 있습니다.

여기서 주요 활용 사례는 오프체인 상호작용의 판단자 역할을 하는 컨트랙트로, 분쟁이 있을 경우에만
생성되면 됩니다.

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract D {
        uint public x;
        constructor(uint a) {
            x = a;
        }
    }

    contract C {
        function createDSalted(bytes32 salt, uint arg) public {
            // 이 복잡한 수식은 단지 어떻게 주소가 미리 계산될 수 있는지
            // 알려주는 것입니다. 이것은 단지 설명을 위해 있습니다.
            // 실제로는 ``new D{salt: salt}(arg)``만이 필요합니다.
            address predictedAddress = address(uint160(uint(keccak256(abi.encodePacked(
                bytes1(0xff),
                address(this),
                salt,
                keccak256(abi.encodePacked(
                    type(D).creationCode,
                    arg
                ))
            )))));

            D d = new D{salt: salt}(arg);
            require(address(d) == predictedAddress);
        }
    }

.. 경고::

    솔트 생성과 관련하여 몇 가지 특이한 점이 있습니다. 컨트랙트는 컨트랙트가 파기된 이후에
    같은 주소에 재생성될 수 있습니다. 비록 생성된 바이트코드가 동일할지라도(이것은 필수사항입니다.
    그렇지 않으면 주소가 변경될 것입니다), 새로 생성된 컨트랙트가 다른 배포된 바이트코드를 가지는 것은 가능합니다 
    이는 생성자가 저장되기 전에 두 생성 간에 변경되었을 수 있는 외부 상태를 조회하여 배포된 바이트코드에
    통합할 수 있기 때문입니다.

수식 코드(Expressions) 실행의 순서
==================================

수식의 실행 순서는 정해지지 않았습니다(정확하게는, 수식 코드 트리에서 한 노드의 자식이 평가되는 순서가
정해지지 않았지만, 당연히 노드 자체보다 먼저 평가됩니다.). 
코드(ststement)가 순서대로 실행되고 불리언 수식 코드에 대한 단락이 수행됨을 보장할 뿐입니다.
.. index:: ! assignment

할당
==========

.. index:: ! assignment;destructuring

할당 파기 및 여러개 값 리턴하기
-------------------------------------------------------

솔리디티는 내부적으로 튜플 타입, 즉 컴파일 시점에서 숫자가 상수인 잠재적으로 다른
유형의 객체 목록을 허용합니다. 이러한 튜플은 동시에 여러개 값을 리턴하는 데에 쓰일 수
있습니다. 그런 다음 새로 선언된 변수나 기존에 존재하던 변수(또는 일반적인 LV값)에 할당할 수 있습니다.

튜플은 솔리디티에서 적절한 타입은 아니고, 코드의 구문적 형태를 위해 사용될 뿐입니다.

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;

    contract C {
        uint index;

        function f() public pure returns (uint, bool, uint) {
            return (7, true, 2);
        }

        function g() public {
            // 변수들은 타입과 함께 선언되고 리턴 튜플로부터 할당됩니다.
            // 모든 요소를 명시할 필요 없습니다(단, 숫자는 일치해야 합니다.)
            (uint x, , uint y) = f();
            // 값 위치를 바꾸는 흔한 방법 -- non-value 저장 타입에는 적용이 안됩니다. 
            (x, y) = (y, x);
            // 구성요소를 (변수 선언의 경우에도)제외할 수 있습니다.
            (index, , ) = f(); // 인덱스를 7로 지정합니다.
        }
    }

변수 선언과 선언이 아닌 할당을 혼합할 수 없습니다. 즉, ``(x, uint y) = (1, 2);``는
유효하지 않습니다.

.. note::

    0.5.0 이전에는 튜플에 왼쪽부터 채우거나 오른쪽부터 채우는 등
    더 작은 사이즈로 할당하는 것이 가능했습니다(심지어 비었더라도). 지금은 허용되지 않으니,
    양측은 같은 구성요소 개수를 가져야 합니다.

.. 경고::

    참조 유형이 포함된 경우 여러 번수에 동시에 할당할 때 예기치 않은 카피 동작을 
    유발할 수 있으므로 조심해 주십시오.

배열과 구조체의 복잡성
------------------------------------

할당은 ``bytes`` and ``string``를 포함하는 배열이나 구조와 같은 non-value 타입의 경우 더복잡합니다.
자세한 내용은 :ref:`Data location and assignment behaviour <data-location-assignment>`을 참조하세요.

아레 예제에서 ``g(x)``에 대한 호출은 ``x``에 효과가 없습니다. 왜냐하면 메모리에 독립적인 저장값의 복사본을
생성하기 때문입니다. 하지만, ``h(x)``는 복사본이 아닌 참조값만 전달되므로 성공적으로 변경합니다

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    contract C {
        uint[20] x;

        function f() public {
            g(x);
            h(x);
        }

        function g(uint[20] memory y) internal pure {
            y[2] = 3;
        }

        function h(uint[20] storage y) internal {
            y[3] = 4;
        }
    }

.. index:: ! scoping, declarations, default value

.. _default-value:

범위 지정(Scoping) 및 선언
========================

선언된 변수는 바이트 표현으로 모두 0인 초기 기본값을 가지게 될 것입니다.
변수의 "기본값"은 어떤 형이든 일반적인 "0 상태"입니다.
예를 들어, ``bool``의 기본값은 ``false``입니다. ``uint`` 또는 ``int``의 기본값은 ``0``입니다.
정적 크기 배열과 ``bytes1`` ~ ``bytes32``의 경우, 각각의 독립된 요소들은 각 타입에 해당하는 기본값으로 초기화됩니다.
``bytes``와 ``string``같은 동적 크기 배열의 경우, 기본값은 문자열의 빈 배열(empty array of string)입니다.
``enum``타입의 기본값은 첫 멤버의 타입을 따라갑니다.

솔리디티의 범위 지정은 C99(와 많은 다른 언어들)의 널리 퍼진 범위 지정 규칙을 따릅니다:
변수는 선언 직후의 시점부터 선언문을 포함하는 가장 작은 ``{ }``-블록의 끝까지 사용할 수 있습니다.
이 규칙에 대한 예외로 for-루프의 초기화 부분에서 선언된 변수는 for-루프가 끝날 때까지만 사용할 수 있습니다.

파라미터와 비슷한 변수(함수 파라미터, 변경자 파라미터, 캐치 파라미터)는 
다음과 같은 코드 블럭 내부에서 사용할 수 있습니다 - 함수, 변경자 파라미터에 대한 함수/변경자의 본문과
캐치 파라미터에 대한 캐치 블록.

코드 블럭 바깥, 예제 함수, 컨트랙트, 사용자 정의 타입 등등이 선언된 변수와 다른 아이템들은,
선언되지 전에도 사용할 수 있습니다. 즉 상태 변수가 선언된 후 함수 호출이 재귀적으로 호출하기 전에
상태 변수를 사용할 수 있습니다. 

결과적으로, 다음 예제의 경우 두 변수가 같은 이름을 갖고 있음에도 다른 범위에 존재하므로
경고 없이 컴파일될 것입니다.

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;
    contract C {
        function minimalScoping() pure public {
            {
                uint same;
                same = 1;
            }

            {
                uint same;
                same = 3;
            }
        }
    }

C99 범위 지정 규칙의 특별한 예로, 다음을 유의하십시오.
``x``에 대한 첫번째 할당은 내부 변수가 아닌 외부 변수로의 할당입니다.

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;
    // 이것은 경고를 보고할 것입니다.
    contract C {
        function f() pure public returns (uint) {
            uint x = 1;
            {
                x = 2; // 이것은 외부 변수에 할당됩니다.
                uint x;
            }
            return x; // x는 2의 값을 가집니다.
        }
    }

.. 경고::

    솔리디티 0.5.0 이전 버전은 JavaScript와 같은 범위 지정 규칙을 따라갔습니다.
    즉 함수 내의 어디에서나 선언된 변수는 그것이 선언된 위치와 관계없이 전체 함수의 범위에 있었습니다.
    다음 예제에서는 컴파일에 사용되었지만 0.5.0 로 시작하는 버전에서 오류가 발생하는 코드 스니팻펫을 보여줍니다.

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;
    // 이 것은 컴파일되지 않습니다.
    contract C {
        function f() pure public returns (uint) {
            x = 2;
            uint x;
            return x;
        }
    }


.. index:: ! safe math, safemath, checked, unchecked
.. _unchecked:

확인되거나 확인되지않은 산술
===============================


오버플로우 또는 언더플로우는 범위가 제한되지 않은 정수에서의 산술 연산의 결과값이 결과 유형의 범위를
벗어나는 상황입니다.

솔리디티 0.8.0 이전에는, 추가 확인을 도입하는 라이브러리의 광범위한 사용으로 이어지는
언더프롤우 또는 오버플로우의 경우 산술 연산이 항상 랩핑되었습니다.

솔리디티 0.8.0부터, 모든 산술 연산은 기본적으로 오버풀로우나 언더플로우로 되돌아갑니다.
따라서 이러한 라이브러리 사용이 불필요해졌습니다.

이전 동작을 얻기 위해, ``unchecked``블록을 사용할 수 있습니다:

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity ^0.8.0;
    contract C {
        function f(uint a, uint b) pure public returns (uint) {
            // 이 뺴기 연산은 언더플로우로 랩핑될 것입니다.
            unchecked { return a - b; }
        }
        function g(uint a, uint b) pure public returns (uint) {
            // 이 빼기 연산은 언더플로우에서 되돌아갈 것입니다.
            return a - b;
        }
    }

``f(2, 3)``은 ``2**256-1``을 반환할 것이고, ``g(2, 3)``는 잘못된 값을 출력합니다.

``unchecked`` 블록은 블록 내부의 모든 곳에서 사용될 수 있지만, 블록의 대체물로 사용할 수는 없습니다.
또한 블록에 대해 중첩될 수 없습니다.

설정은 블록 내부에 구문적으로 존재하는 코드에 대해서만 효과를 가집니다.
``unchecked`` 내에서 호출된 함수는 특성을 상속하지 않습니다.

.. note::
    
    모호함을 피하기 위해, ``unchecked``블럭 내부에서 ``_;``를 사용할 수 없습니다.

다음의 연산자들은 언더플로우와 오버플로우로 인해 실패한 연산을 발생할 수 있으며 ``unchecked`` 블록
내에서 사용되는 경우 오류 없이 랩핑합니다.

``++``, ``--``, ``+``, 바이너리 ``-``, 단일 ``-``, ``*``, ``/``, ``%``, ``**``

``+=``, ``-=``, ``*=``, ``/=``, ``%=``

.. 경고::

    ``unchecked`` 블럭을 사용하여 0으로 나누는 경우를 비활성화하는 것은 불가능합니다.

.. note::

   비트 단위 연산자는 오버플로우나 언더플로우를 확인하지 않습니다. 
   이는 특히 2의 제곱연산에서 정수 나눗셈과 곱셈 대신 비트 시프트 연산 
   (``<<``, ``>>``, ``<<=``, ``>>=``)을 사용할 때 잘 나타납니다.
   예를 들어 ``type(uint256).max * 8``은 되돌아가는 반면 ``type(uint256).max << 3``은 되돌아가지 않습니다.

.. note::

    ``int x = type(int).min; -x;``의 두 번쨰 문장은 음의 범위가 양의 범위보다 하나 더 많은 값을 가질 수 있기 때문에
    오버플로우가 발생갑니다.


명시적 타입 변환은 항상 생략하고, 정수에서 열거형으로 변환하지 않는 경우를 제외하고
실패한 연산을 야기하지 않을 것입니다.

.. index:: ! exception, ! throw, ! assert, ! require, ! revert, ! errors

.. _assert-and-require:

에러 처리: 검증, 요구, 되돌리기과 예외
======================================================

솔리디티는 에러를 처리하기 위해 상태-되돌리기 예외를 사용합니다.
이러한 예외는 현재 호출(과 그것의 모든 하위 호출)에서 상태에 대한 모든 변경사항을 취소하고
호출자에게 오류를 표시합니다.

하위 호출에서 예외가 발생했을 때, "버블 업" (즉, 예외가 다시 던져짐)은 ``try/catch`` 구문에
잡히지 않는 한 자동으로 발생합니다. 이 규칙의 예외는 ``send``와 하위 레벨 함수 ``call``, ``delegatecall`` 그리고
``staticcall``입니다 : 이들은 "버블 업" 대신  ``false``을 첫 번째 반환 값으로 반환합니다.

.. 경고::

    하위 레벨 함수 ``call``, ``delegatecall`` 그리고 ``staticcall``는 EVM 설게의 일환으로
    호출된 계정이 존재하지 않는 경우 첫 번쨰 리턴값으로 ``true`` 를 리턴합니다.
    계정의 존재는 함수 호출이 필요하다면 호출 이전에 무조건 확인되어야 합니다.

예외는 호출자에게 다시 전달되는 에러 데이터를 :ref:`error instances <errors>`의 형태로
포함할 수 있습니다.빌트인 에러인 ``Error(string)``와 ``Panic(uint256)``는 아래 설명과 같이 특수한 함수에
의해 사용됩니다. ``Error``는 일반적인 오류 조건에 사용되는 반면 ``Panic``은 버그가 없는 코드에 존재해서는 안 되는 오류에서
사용됩니다.

``assert``를 통한 패닉(Panic)과 ``require``를 통한 에러(Error)
----------------------------------------------

편의 함수 ``assert``와 ``require``는 조건이 충족되지 않았다면 조건 확인과 예외 호출을 위해 사용될 수 있습니다.

``assert``함수는 ``Panic(uint256)``타입의 에러를 생성합니다.
위와 같은 에러는 컴파일러에 의해 아래의 특정한 상황들에 의해 생성됩니다.

'Assert'는 내부 오류 테스트 및 불변성 검사에만 사용되어야 합니다. 제대로 작동하는 코드는
잘못된 외부 입력에 대해서도 패닉을 생성해야 합니다. 이런 일이 발생하면, 계약에 수정
해야하는 오류가 발생합니다. 언어 분석 도구는 계약을 평가하여 패닉을 발생시키는 조건과 함수 호출을 식별할 수 있습니다.

패닉 예외는 다음과 같은 상황에서 발생됩니다.
에러 데이터와 함꼐 제공된 에러 코드는 패닉의 한 종류를 나타냅니다.

#. 0x00: 일반적인 컴파일러에 삽입된 패닉에 사용됩니다.
#. 0x01: 거짓이라고 평가하는 인수로 'assert'를 호출할 때
#. 0x11: 산술 연산으로 인해 ``unchecked { ... }``블록 외부에서 언더플로우 또는 오버플로우가 발생할 때
#. 0x12; 0으로 나눌 때(예. ``5 / 0`` 또는 ``23 % 0``).
#. 0x21: 너무 크거나 음수값을 열겨형으로 변환할 때
#. 0x22: 잘못 인코딩된 저장소 바이트 배열에 접근할 때
#. 0x31: 빈 배열에 ``.pop()``을 호출할 때
#. 0x32: 배열에 접근할 때, ``bytesN`` 또는 범위를 벗어나는 인덱스 또는 음수 인덱스의 배열 슬라이스 사용(예. ``x[i]`` 인데 ``i >= x.length`` 또는 ``i < 0``).
#. 0x41: 메모리를 너무 많이 할당하거나 너무 큰 배열을 생성할 때
#. 0x51: 내부 함수 타입의 0-초기화 변수를 호출할 때

``require`` 함수는 데이터가 없는 에러를 만들거나 ``Error(string)`` 타입의 오류를 발생시킵니다.
실행 시간까지 감지할 수 없는 유효한 조건을 보장하기 위해 사용해야 합니다.
여기에는 입력 값 또는 외부 계약에서 호출까지의 리턴 값에 대한 조건이 포함됩니다.


.. note::

    현재 사용자 지정 오류를 ``require``와 함께 사용할 수 없습니다. 
    ``if (!condition) revert CustomError();``을 대신 사용해주세요.


다음과 같은 상황에서 ``Error(string)`` 예외(또는 데이터가 없는 예외)가 
컴파일러에 의해 발생합니다.


#. ``x``가 ``false``일 때 ``require(x)``를 호출하는 경우.
#. ``revert()`` 또는 ``revert("description")``를 사용하는 경우.
#. 코드가 포함되지 컨트랙트를 대상으로 외부 함수 호출을 수행하는 경우.
#. 컨트랙트가 ``payable``변경자 (생성자와 폴백 함수 포함) 없이 퍼블릭 함수를 통해 이더를 받는 경우
#. 컨트랙트가 공개 게터 함수를 통해 이더를 받는 경우.

다음의 경우, 외부 호출(제공된 경우) 에러 데이터가 전달됩니다.
즉, `Error` 또는 `Panic`(또는 제공된 다른 어떤 것)이 유발될 수 있음을 의미합니다.

#. ``.transfer()`` 가 실패한 경우.
#. 낮은 레벨의 연산 ``call``, ``send``, ``delegatecall``, ``callcode`` 또는 ``staticcall``를 제외하고 
   메세지 호출을 통해 함수를 호출했지만 정상적으로 종료되지 않은 경우
   (예 : 가스가 부족하거나, 일치하는 함수가 없거나, 예외를 발생시키는 경우).
   낮은 레벨의 연산은 예외를 던지지 않고  ``false``를 반환함으로써 실패를 표현합니다.
#. 컨트랙트를 ``new``키워드를 사용하여 생성하였지만 컨트랙트 생성이 적절히 수행되지 않은 경우.
   :ref:`does not finish properly<creating-contracts>`

선택적으로 ``require``에는 메세지 문자열을 제공할 수 있지만, ``assert``에는 불가합니다.

.. note::

    문자열 인수를 ``require``에 제공하지 않으면, 에러 셀렉터마저 포함하지 않고 빈 에러 데이터로 되돌아갑니다


다음의 예제는 입력의 조건을 표현하기 위해 ``require``를 어떻게 쓰는지,
내부 에러 확인을 확인하기 위해 ``assert``를 어떻게 쓰는지 보여줍니다. 

.. code-block:: solidity
    :force:

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;

    contract Sharer {
        function sendHalf(address payable addr) public payable returns (uint balance) {
            require(msg.value % 2 == 0, "Even value required.");
            uint balanceBeforeTransfer = address(this).balance;
            addr.transfer(msg.value / 2);
            // 전송이 실패에 대한 예외를 던지고 여기로 다시 콜백할 수 없기
            // 때문에, 우리가 여전히 절반의 돈을 가질 수 있는 방법이 없습니다.
            assert(address(this).balance == balanceBeforeTransfer - msg.value / 2);
            return address(this).balance;
        }
    }


내부적으로 솔리디티는 되돌리기 연산을 수행합니다(명령어 ``0xfd``). 이는
EVM이 모든 변경사항을 상태로 되돌리는 것을 야기합니다. 되돌리는 이유는
기대했던 효과가 일어나지 않았기 떄문에, 실행을 계속할 안전한 방법이 없기 때문입니다.
거래의 원자성을 유지하고 싶기 때문에, 가장 안전한 행위는 모든 변경사항을 되돌리고 모든 거래 효과를(최소한 호출)
제거하는 것입니다.

두 경우 모두 호출자는 이러한 문제에 대해 ``try``/``catch``를 사용하여 대응할 수 있지만,
피호출자의 변경사항은 항상 되돌아갈 것입니다.

.. note::

    솔리디티 0.8.0 이전에 패닉 예외는 호출에 사용할 수 있는 모든 가스를 소비하는
    ``invalid`` 명령 코드를 사용하곤 했습니다. ``require``를 사용하는 예외는 메트로폴리스가
    나오기 전까지는 모든 가스를 소비했습니다. 


.. _revert-statement:

``revert``
----------

직접적인 되돌리기는  ``revert`` 구문과  ``revert``함수를 통해 실행될 수 있습니다.

``revert``구문은 괄호 없이 사용자 정의 오류를 직접 인수로 사용합니다.

    revert CustomError(arg1, arg2);

역호환성의 이유로, 괄호를 사용하여 문자열을 받아들이는 ``revert()`` 함수도 있습니다.

    revert();
    revert("description");

에러 데이터는 호출자에게 재전달되어 그곳에 붙잡혀 있을 수 있습니다.
``revert()``를 사용하는 것은 에러 데이터 없이 반환되는 반면 ``revert("description")``는
``Error(string)``를 생성합니다.

사용자 정의 에러 인스턴스를 사용하는 것은 일반적으로 문자열 설명보다 훨씬 저렴한데,
오류를 설명하기 위해 이름을 사용할 수 있으며, 이는 오직 4바이트로 인코딩되기 때문입니다.
더 긴 설명은 NatSpec을 통해 제공할 수 있으므로 비용을 유발하지 않습니다.

다음 예제는 에러 문자열과 사용자 정의 에러 인스턴스를 ``revert``및 이와 동등한
``require``와 함께 사용하는 방법을 보여줍니다:

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity ^0.8.4;

    contract VendingMachine {
        address owner;
        error Unauthorized();
        function buy(uint amount) public payable {
            if (amount > msg.value / 2 ether)
                revert("Not enough Ether provided.");
            // 대체 방법
            require(
                amount <= msg.value / 2 ether,
                "Not enough Ether provided."
            );
            // 구입을 수행합니다.
        }
        function withdraw() public {
            if (msg.sender != owner)
                revert Unauthorized();

            payable(msg.sender).transfer(address(this).balance);
        }
    }

``if (!condition) revert(...);``와 ``require(condition, ...);`` 두 가지 방법은
``revert`` 와 ``require`` 인수가 부작용이 없는 한 동등합니다.
그들이 단지 문자열일 때를 예로 들 수 있습니다.

.. note::

    ``require`` 함수는 다른 어떤 함수와 마찬가지로 평가됩니다.
    이는 함수 자체가 실행되기 전 모든 인수가 평가된다는 것을 의미합니다. 특히
    ``require(condition, f())``에서 ``f``함수는 ``condition``이 ture일 때도 실행됩니다.

제공된 문자열은 함수 ``Error(string)``에 대한 호출인 것처럼 :ref:`abi-encoded <ABI>`입니다.
위의 예제에서 ``revert("Not enough Ether provided.");``는 다음 16진수를 에러 리턴 데이터로 리턴합니다.
.. code::

    0x08c379a0                                                         // Error(string)을 위한 함수 셀렉터
    0x0000000000000000000000000000000000000000000000000000000000000020 // 데이터 오프셋
    0x000000000000000000000000000000000000000000000000000000000000001a // 문자열 길이
    0x4e6f7420656e6f7567682045746865722070726f76696465642e000000000000 // 문자열

제공된 메세지는 아래에서 보여주듯 ``try``/``catch``를 사용하여 호출자에 의해 되찾아올 수 있습니다.

.. note::

    예전에는 0.4.13 버전에서 축소되고 0.5.0 버전에서 삭제된 ``revert()``와 같은 기능을 수행하는
    ``throw``라고 불리는 키워드가 있었습니다. 


.. _try-catch:

``try``/``catch``
-----------------

외부 호출의 실패는 다음과 같이 try/catch 구문을 사용하여 잡을 수 있습니다:

.. code-block:: solidity

    // SPDX-라이센스-식별자: GPL-3.0
    pragma solidity >=0.8.1;

    interface DataFeed { function getData(address token) external returns (uint value); }

    contract FeedConsumer {
        DataFeed feed;
        uint errorCount;
        function rate(address token) public returns (uint value, bool success) {
            // 10개 이상의 에러가 있는 경우
            // 영원히 매커니즘을 비활성화합니다
            require(errorCount < 10);
            try feed.getData(token) returns (uint v) {
                return (v, true);
            } catch Error(string memory /*reason*/) {
                // revert가 getData 내부에서 호출되고
                // reason 문자열이 제공된 경우
                // 실행됩니다.
                errorCount++;
                return (0, false);
            } catch Panic(uint /*errorCode*/) {
                // 패닉, 즉 0으로 나눔 또는 오버플로우와 같은 
                // 심각한 에러의 경우 실행됩니다. 에러 코드는
                // 에러의 종류를 결정하기 위해 사용됩니다.
                errorCount++;
                return (0, false);
            } catch (bytes memory /*lowLevelData*/) {
                // revert() 가 사용되는 경우 실행됩니다.
                errorCount++;
                return (0, false);
            }
        }
    }

``try``키워드는 외부 함수 호출 또는 컨트랙트 생성을 나타내는 (``new ContractName()``)가 뒤따라야 합니다.
표현식 내부의 에러는 잡히지 않으며(예를 들어 내부 함수 호출 또한 포함하는 복잡한 표현식의 경우),
오직 외부 함수 자체 내에서 발생하는 되돌리기만 발생합니다. 뒤에 오는 ``returns`` 부분(선택 사항)은
외부 호출에 의해 리턴된 유형과 일치하는 리턴 변수를 선언합니다.
오류가 없었을 경우, 이 변수들은 할당되고 컨트랙트의 실행은 첫 번째 성공 블럭 내부에서 진행됩니다.
만약 성공 블록의 끝에 도달하면, ``catch``블록 후에 실행이 계속됩니다.

솔리디티는 에러 타입에 대해 여러 종류의 캐치 블록을 지원합니다:

- ``catch Error(string memory reason) { ... }``: 이 캐치 구문은 에러가 
   ``revert("reasonString")`` 또는 ``require(false, "reasonString")``
   (또는 이러한 예외를 발생시키는 내부 에러)로 인해 발생한 경우 실행됩니다.

- ``catch Panic(uint errorCode) { ... }``: 이 캐치 구문은 에러가 패닉, 즉 0으로 나눔, 유효하지
  않은 배열 접근, 산술적 오버플로우 및 그 외로 인해 발생한 경우 실행됩니다.

- ``catch (bytes memory lowLevelData) { ... }``: 이 구문은 에러 서명이 다른 구문과 일치하지 않는
  경우, 에러를 디코딩하는 동안 에러가 발생하는 경우, 예외가 포함된 에러 데이터가 제공되지
  않은 경우 실행됩니다. 이 경우 선언된 변수는 낮은 레벨의 데이터에 대한 접근을 제공합니다. 

- ``catch { ... }``: 오류 데이터에 관심이 없는 경우, 이전 구문들 대신
  ``catch { ... }`` 만(유일 캐치 구문으로) 사용하셔도 됩니다.

향후 다른 타입의 에러 데이터를 지원할 예정입니다.
``Error`` 와 ``Panic`` 문자열은 현재 그대로 parsed되며 식별자로 취급되지 않습니다.

모든 에러 케이스를 확인하기 위해, 최소한 ``catch { ...}`` 또는 
``catch (bytes memory lowLevelData) { ... }`` 구문이 있어야 합니다.

``returns``과 ``catch``구문에 선언된 변수는 따라오는 블록의 범위 내에 있습니다.

.. note::

    try/catch-statement 내부에서 리턴 데이터를 디코딩하는 동안 에러가 발생하면,
    현재 실행 중인 계약에서 예외가 발생하고, 이 때문에 캐치 구문에 의해 잡히지 않습니다.
    만약 ``catch Error(string memory reason)``의 디코딩 중에 에러가 발생하여 낮은 레벨의
    캐치 구문이 있으면, 에러가 그 곳에 잡힙니다.

.. note::

    실행이 캐치 블록에 도달하면, 외부 호출의 상태-변경 효과가 되돌려집니다.
    실행이 성공 블럭에 도달하면, 효과는 되돌려지지 않습니다.
    효과가 되돌려진 경우 실행은 캐치 블록에서 계속 실행되거나  try/catch 구문의
    실행이 되돌려집니다(예를 들어 위에서 언급한 대로 디코딩 실패로 인해 또는 
    낮은 레벨의 캐치 구문을 제공하지 않기 때문에).

.. note::

    호출 실패 이유는 다양할 수 있습니다. 에러 메세지가 호출된 컨트랙트에서 직접
    전송된다고 가정하지 마십시오:
    에러는 호출 체인 아래에서 더 깊이 발생했을 수 있으며, 호출 컨트랙트가 방금 전달되었을 수 있습니다.
    또한, 이는 고의적인 에러 상황이 아닌 가스 부족 상황일 수도 있습니다:
    호출자는 항상 호출 시 최소 1/64 만큼의 가스를 보유하고 있으므로 컨트랙트 호출에 가스가 빠져니기도
    호출자는 여전히 약간의 가스가 남아있습니다.
