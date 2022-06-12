.. _formal_verification:

##################################
SMTChecker와 수식 검증 (Formal Verification)
##################################

Formal verification을 사용하면 당신의 소스 코드가 특정 수식에 대한 사양을 만족하는지 
자동화된 수학적 검증을 수행할 수 있습니다. The specification is still formal (just
as the source code), but usually much simpler.

Note that formal verification itself can only help you understand the
difference between what you did (the specification) and how you did it
(the actual implementation). You still need to check whether the specification
is what you wanted and that you did not miss any unintended effects of it.

Solidity는 공식 검증 접근법을 `SMT (Satisfiability Modulo Theories) 
<https://en.wikipedia.org/wiki/Satistiability_modulo_theories>`_ 와
`Horn <https://en.wikipedia.org/wiki/Horn-satistiability>`_ 를 이용하여 구현합니다.
SMTChecker 모듈은 자동적으로 해당 코드가 ``require``과 ``assert`` 에 의해 제시된
사양을 만족하는지 증명을 시도합니다. 즉, ``require``구문은 가정으로 간주하고 ``assert``구문
안의 조건이 항상 참임을 증명하려고 합니다. 만약 assertion failure가 발견되었을 경우,
해당 assertion이 어떻게 위반될 수 있는지 보여주는 반례가 사용자에게 제공될 수 있습니다.
만약 no warning이 property에 대해 SMTChecker로부터 제공되었다면, 이는 해당 property가 안전하다는
것을 의미합니다.

SMTChecker가 컴파일 타임에 확인하는 검증 타겟은 다음과 같습니다:

- 산술적 언더플로우와 오버플로우.
- 0으로 나누기.
- 쓸모없는 조건과 도달할 수 없는 코드.
- 비어있는 배열에서의 Pop 행위.
- 경계를 벗어난 인덱스의 접근.
- 송금을 위한 자금의 부족.

위에 나열된 모든 타겟들은 만약 모든 엔진이 허용되어 있다면 Solidity >=0.8.7 버전에서의 언더플로우와 오버플로우를 제외하고 기본적으로 자동으로 검증됩니다.

SMTChecker가 보고하는 잠재적인 경고들은 다음과 같습니다:

- ``<failing  property> happens here.``. 이 경고의 의미는 SMTChecker가 특정한 property가 실패했음을 입증함을 의미합니다. 반례가 있을 수 있지만, 복잡한 상황들에서는 반례에 대해 표시하지 않을 수도 있습니다. 이 결과는 또한 SMT encoding이 표현하기 어렵거나 불가능한 Solidity 코드에 대한 추상화를 추가하는 특정한 케이스들에 경우 False positive일 수도 있습니다.

- ``<failing property> might happen here``. 이 경고의 의미는 solver가 주어진 시간 초과 내에 두 경우 모두를 증명하지 못했음을 의미합니다. 결과를 알 수 없기 때문에, SMTChecker는 건전성(Soundness)에 대한 잠재적 실패를 리포트합니다. 이 경우 쿼리 시간 초과를 늘려 해결할 수 있기는 하지만, 엔진이 이를 해결하기에 문제 자체가 너무 어려울 수도 있습니다.

SMTChecker를 활성화하기 위해서, 당신은 :ref:`which engine should run<smtchecker_engines>`를 선택해야 하며 여기서 기본 값은 no engine (엔진 없음) 입니다. 엔진을 선택하면 모든 파일에서 SMTChecker가 활성화됩니다.

.. note::

    Solidity 0.8.4버전 이전에는, SMTChecker를 활성화하기 위한 기본적인 방법은
    ``pragma experimental SMTChecker;``를 통한 방법이었고 오직 pragma가 포함된
    컨트랙트만 분석되었습니다. 해당 pragma는 더 이상 사용되지 않으며 이전 버전과의
    호환성을 위해 SMTChecker를 계속 활성화하지만 Solidity 0.9.0에서 제거되었습니다.
    또한 참고로 현재는 단일 파일에서도 pragma를 사용하면 모든 파일에 대해 SMTChecker를
    사용할 수 있습니다.

.. note::

    검증 대상에 대한 경고가 없다는 것은 SMTChecker 및 기본 솔버에 버그가 없다고 가정할 때,
    정확성에 대한 수학적 검증에 논쟁의 여지가 없다는 것을 나타냅니다. 일반적인 경우에 이러한
    문제들을 자동으로 해결하는 것은 *매우 어렵거나* 혹은 종종 *불가능* 하다는 것을 명심해야합니다.
    따라서, 몇몇의 property들은 해결되지 않거나 또는 매우 큰 계약들에서 false positive(오탐)에 
    이를 수 있습니다. 모든 입증된 property들은 중요한 성과로 간주되어야 합니다. 숙련된 유저들은 더 복잡한 property들을 증명하기 위해 도움을 줄 수 있는 몇 가지 옵션을 추가로 배우려면 :ref:`SMTChecker Tuning <smtchecker_options>`를 보는 것이 도움이 될 것입니다.


********
튜토리얼
********

Overflow
========

.. code-block:: Solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    contract Overflow {
        uint immutable x;
        uint immutable y;

        function add(uint _x, uint _y) internal pure returns (uint) {
            return _x + _y;
        }

        constructor(uint _x, uint _y) {
            (x, y) = (_x, _y);
        }

        function stateAdd() public view returns (uint) {
            return add(x, y);
        }
    }

위의 컨트랙트 예제는 오버플로우를 확인하는 예제를 보여주고 있습니다.
SMTChecker는 Solidity 버전이 0.8.7 버전 이상일 경우에는 기본적으로 언더플로우와 오버플로우를
확인하지 않습니다.
따라서 우리는 커맨드라인 옵션인 ``--model-checker-targets "underflow,overflow"``를 사용하거나,
혹은 JSON 옵션인 ``settings.modelChecker.targets = ["underflow", "overflow"]``를 사용해야 합니다.
:ref:`this section for targets configuration<smtchecker_targets>`를 참고하십시오.
여기에 해당 리포트가 제시되었습니다:

.. code-block:: text

    Warning: CHC: Overflow (resulting value larger than 2**256 - 1) happens here.
    Counterexample:
    x = 1, y = 115792089237316195423570985008687907853269984665640564039457584007913129639935
     = 0

    Transaction trace:
    Overflow.constructor(1, 115792089237316195423570985008687907853269984665640564039457584007913129639935)
    State: x = 1, y = 115792089237316195423570985008687907853269984665640564039457584007913129639935
    Overflow.stateAdd()
        Overflow.add(1, 115792089237316195423570985008687907853269984665640564039457584007913129639935) -- internal call
     --> o.sol:9:20:
      |
    9 |             return _x + _y;
      |                    ^^^^^^^

만약 우리가 ``require`` 구문을 해당 걸러내진 오버플로우 케이스들에 대해 추가한다면,
SMTChecker는 아무런 오버플로우가 도달될 수 없다는 것을 증명합니다. (경고가 리포트되지 않았을 경우):

.. code-block:: Solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    contract Overflow {
        uint immutable x;
        uint immutable y;

        function add(uint _x, uint _y) internal pure returns (uint) {
            return _x + _y;
        }

        constructor(uint _x, uint _y) {
            (x, y) = (_x, _y);
        }

        function stateAdd() public view returns (uint) {
            require(x < type(uint128).max);
            require(y < type(uint128).max);
            return add(x, y);
        }
    }


Assert
======

assertion은 코드의 불변성을 나타냅니다: *모든 입력 및 저장 값을 포함한 모든 트랜잭션*에 대해 true여야
하는 property입니다. 그렇지 않을 경우 버그가 있습니다.

아래에 제시된 코드는 오버플로가 없음을 보장하는 함수 ``f``를 정의합니다.
함수 ``inv``는 ``f``가 단조롭게 증가함을 나타내는 명세를 정의합니다:
가능한 모든 쌍 ``(_a, _b)``에 대해 ``_b > _a``이면 ``f(_b) > f(_a)``입니다.
``f``가 실제로 단조롭게 증가하기 때문에 SMTChecker는 우리의 property가 옳다는 것을 증명합니다.
You are encouraged to play with the property and the function
definition to see what results come out!

.. code-block:: Solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    contract Monotonic {
        function f(uint _x) internal pure returns (uint) {
            require(_x < type(uint128).max);
            return _x * 42;
        }

        function inv(uint _a, uint _b) public pure {
            require(_b > _a);
            assert(f(_b) > f(_a));
        }
    }

우리는 또한 루프 안에 좀 더 복잡한 property들을 검증하기 위해 assertion들을 추가할 수 있습니다.
다음 코드는 제한이 없는 숫자 배열의 최대 요소를 검색하고 발견된 요소가 배열의 모든 요소보다
크거나 같아야 한다는 property를 단언합니다.

.. code-block:: Solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    contract Max {
        function max(uint[] memory _a) public pure returns (uint) {
            uint m = 0;
            for (uint i = 0; i < _a.length; ++i)
                if (_a[i] > m)
                    m = _a[i];

            for (uint i = 0; i < _a.length; ++i)
                assert(m >= _a[i]);

            return m;
        }
    }

이 예에서 SMTChecker는 자동으로 3가지의 property들을 증명하고자 시도합니다:

1. 첫 번째 루프 내의 ``++i`` 는 오버플로우 되지 않는다.
2. 두 번째 루프 내의 ``++i`` 는 오버플로우 되지 않는다.
3. assertion은 항상 참이다.

.. note::

    The properties involve loops, which makes it *much much* harder than the previous
    examples, so beware of loops!

모든 property들은 전부 안전이 올바르게 입증되었습니다. 다른 결과를 보고 싶다면 자유롭게 
프로퍼티를 변경하거나 배열에 제한을 추가하십시오.
예를 들면, 코드를 다음과 같이 변경한 경우

.. code-block:: Solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    contract Max {
        function max(uint[] memory _a) public pure returns (uint) {
            require(_a.length >= 5);
            uint m = 0;
            for (uint i = 0; i < _a.length; ++i)
                if (_a[i] > m)
                    m = _a[i];

            for (uint i = 0; i < _a.length; ++i)
                assert(m > _a[i]);

            return m;
        }
    }

우리에게 다음과 같은 결과를 줍니다:

.. code-block:: text

    Warning: CHC: Assertion violation happens here.
    Counterexample:

    _a = [0, 0, 0, 0, 0]
     = 0

    Transaction trace:
    Test.constructor()
    Test.max([0, 0, 0, 0, 0])
      --> max.sol:14:4:
       |
    14 |            assert(m > _a[i]);


State Properties
================

지금까지의 예제들은 특정 작업이나 알고리즘에 대한 프로퍼티들을 증명하는 
퓨어 코드에 대한 SMTChecker의 사용만을 보여주었습니다.
스마트 컨트랙트 내의 프로퍼티들에 대한 흔한 유형은 컨트랙트의 상태에 연관된 프로퍼티들입니다.
그러한 프로퍼티에 대한 어설션이 실패하도록 하기 위해서는 여러 트랜잭션이 필요할 수 있습니다.

예로서, 두 축 위에서 임의의 범위 (-2^128, 2^128 - 1) 의 좌표 범위를 가지는 2D 격자를 고려해봅시다.
(0, 0) 위치에 로봇을 배치한다고 해봅시다. 그 로봇은 오직 대각선으로만 움직일 수 있으며, 한 번에 한 스텝만,
그리고 격자의 바깥 영역으로 움직일 수 없습니다. 이 로봇의 상태 머신은 아래와 같은 스마트 컨트랙트에 의해
표현될 수 있습니다.

.. code-block:: Solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    contract Robot {
        int x = 0;
        int y = 0;

        modifier wall {
            require(x > type(int128).min && x < type(int128).max);
            require(y > type(int128).min && y < type(int128).max);
            _;
        }

        function moveLeftUp() wall public {
            --x;
            ++y;
        }

        function moveLeftDown() wall public {
            --x;
            --y;
        }

        function moveRightUp() wall public {
            ++x;
            ++y;
        }

        function moveRightDown() wall public {
            ++x;
            --y;
        }

        function inv() public view {
            assert((x + y) % 2 == 0);
        }
    }

함수 ``inv``는 ``x + y``가 짝수가 되어야 하는 상태 머신에 상태 머신의 불변성을 나타냅니다.
SMTChecker는 얼마나 많은 명령어를 우리가 로봇에 제공했는지 여부와 관계 없이 무한히 많다 하더라도
불변성은 *절대* 실패할 수 없음을 증명합니다. 만일 이에 대해 관심있는 독자라면 이 사실을
수동으로 증명하는 것도 가능합니다. Hint: 이 불변성은 귀납적입니다.

우리는 또한 SMTChecker를 속이는 것으로 도달할 수 있다고 생각되는 특정 위치에 대한
경로를 제공하는 것도 가능합니다. 우리는 다음 함수를 추가함으로 (2, 4)는 도달할 수 *없음*을
프로퍼티로서 추가할 수 있습니다.

.. code-block:: Solidity

    function reach_2_4() public view {
        assert(!(x == 2 && y == 4));
    }

이 프로퍼티가 false이며 해당 프로퍼티가 false라는 것을 증명하는 과정 중에,
SMTChecker는 우리에게 정확하게 *어떻게* (2, 4)에 도달할 수 있는지에 대해 설명합니다:

.. code-block:: text

    Warning: CHC: Assertion violation happens here.
    Counterexample:
    x = 2, y = 4

    Transaction trace:
    Robot.constructor()
    State: x = 0, y = 0
    Robot.moveLeftUp()
    State: x = (- 1), y = 1
    Robot.moveRightUp()
    State: x = 0, y = 2
    Robot.moveRightUp()
    State: x = 1, y = 3
    Robot.moveRightUp()
    State: x = 2, y = 4
    Robot.reach_2_4()
      --> r.sol:35:4:
       |
    35 |            assert(!(x == 2 && y == 4));
       |            ^^^^^^^^^^^^^^^^^^^^^^^^^^^

(2, 4)에 도달할 수 있는 다른 경로가 존재하기 때문에 위의 경로가 반드시
결정적인 것만은 아닙니다. 표시되는 경로의 선택은 사용된 솔버, 해당 버전 또는
무작위로 변경될 수 있습니다.


External Calls and Reentrancy
=============================

모든 외부 호출은 SMTChecker에 의해 알려지지 않은 코드를 호출하는 것으로 간주될 수 있습니다.
그 이유는 호출된 컨트랙트의 코드가 컴파일 시간에 사용 가능하더라도, 배포된 컨트랙트가 컴파일 시간에
인터페이스가 제공된 컨트랙트와 실제로 동일하다는 보장이 없기 때문입니다.

일부 경우에는, 외부에서 호출된 코드가 호출자의 컨트랙트 재작성을 포함하여 무엇이든 할 수 있는
경우에도 여전히 참인 상태 변수에 대한 속성을 자동으로 유추하는 것이 가능합니다.

.. code-block:: Solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    interface Unknown {
        function run() external;
    }

    contract Mutex {
        uint x;
        bool lock;

        Unknown immutable unknown;

        constructor(Unknown _u) {
            require(address(_u) != address(0));
            unknown = _u;
        }

        modifier mutex {
            require(!lock);
            lock = true;
            _;
            lock = false;
        }

        function set(uint _x) mutex public {
            x = _x;
        }

        function run() mutex public {
            uint xPre = x;
            unknown.run();
            assert(xPre == x);
        }
    }

위에 제공된 예시는 재진입을 금지하기 위한 뮤텍스 플래그를 사용하는 컨트랙트를 보여줍니다.
솔버는 ``unknown.run()``이 호출될 때, 컨트랙트는 이미 "락"된 상태이므로 알 수 없는 것이
무엇이든 관계 없이 ``x``의 값을 변경할 수 없다고 추론할 수 있습니다.

If we "forget" to use the ``mutex`` modifier on function ``set``, the
SMTChecker is able to synthesize the behaviour of the externally called code so
that the assertion fails:

.. code-block:: text

    Warning: CHC: Assertion violation happens here.
    Counterexample:
    x = 1, lock = true, unknown = 1

    Transaction trace:
    Mutex.constructor(1)
    State: x = 0, lock = false, unknown = 1
    Mutex.run()
        unknown.run() -- untrusted external call, synthesized as:
            Mutex.set(1) -- reentrant call
      --> m.sol:32:3:
       |
    32 |        assert(xPre == x);
       |        ^^^^^^^^^^^^^^^^^


.. _smtchecker_options:

*****************************
SMTChecker의 옵션들과 튜닝
*****************************

Timeout
=======

SMTChecker는 솔버별로 선택한 시간과 정확히 관계가 없는 하드코딩된 리소스 제한(``rlimit``)을 사용합니다.
우리는 ``rlimit`` 옵션을 디폴트로 선택하는데 because it gives more determinism guarantees than time inside the solver.

이 옵션은 쿼리당 대략 "a few seconds timeout"으로 변환됩니다. 물론 많은 프로퍼티들은 매우 복잡하며
결정론이 중요하지 않은 곳에서 해결하기 위해 매우 많은 시간이 필요합니다.
만약 SMTChecker가 기본 ``rlimit``으로 컨트랙트 프로퍼티를 해결하지 못하는 경우, 타임아웃은 CLI 옵션인
``--model-checker-timeout <time>`` 또는 JSON 옵션인 ``settings.modelChecker.timeout=<time>``을 통해
밀리 세컨드로 제공될 수 있습니다, 0이 제공된 경우 이는 타임아웃이 없음을 의미합니다.

.. _smtchecker_targets:

검증 대상 (Verification Targets)
====================

검증 대상들의 타입들은 CLI 옵션인 ``--model-checker-target <targets>`` 또는 JSON 옵션인
``settings.modelChecker.targets=<targets>``를 통해 마찬가지로 커스터마이징 될 수 있는 
SMTChecker에 의해 생성될 수 있습니다.
CLI의 경우, ``<targets>``는 공백 없이 쉼표로 구분된 하나 이상의 검증 대상 목록이며,
JSON 입력 문자열로 된 하나 이상의 대상 배열입니다.
대상들을 표현하는 키워드는 다음과 같습니다:

- Assertions: ``assert``.
- Arithmetic underflow: ``underflow``.
- Arithmetic overflow: ``overflow``.
- Division by zero: ``divByZero``.
- Trivial conditions and unreachable code: ``constantCondition``.
- Popping an empty array: ``popEmptyArray``.
- Out of bounds array/fixed bytes index access: ``outOfBounds``.
- Insufficient funds for a transfer: ``balance``.
- All of the above: ``default`` (CLI only).

해당 타겟들에 대한 흔한 서브셋은 다음처럼 제공될 수 있습니다, 예시:
``--model-checker-targets assert,overflow``.

모든 타겟들은 Solidity 버전이 0.8.7 이상인 경우의 오버플로우와 언더플로우를 제외하고 기본적으로 검증됩니다.

검증 대상을 분할하는 방법과 시점에 대한 정확한 휴리스틱은 없습니다,
그러나 특히 대규모의 컨트랙트를 처리할 때 유용할 수 있습니다.

검증되지 않은 대상 (Unproved targets)
================

만약 어떠한 검증되지 않은 대상이 있다면, SMTChecker는 얼마나 많은 검증되지 않은 대상이 있는지에 대한
경고를 표시합니다. 만약 유저가 모든 상세한 검증되지 않은 대상에 대한 모든 정보를 보고 싶다면,
CLI 옵션인 ``--model-checker-show-unproved``와 JSON 옵션인 ``settings.modelChecker.showUnproved = true``
가 사용될 수 있습니다.


검증된 컨트랙트 (Verified Contracts)
==================

기본적으로 지정된 소스의 모든 배포 가능한 컨트랙트들은 배포된 컨트랙트로 별도로 분석됩니다.
이것이 의미하는 바는 만약 컨트랙트가 많은 직접적인 그리고 간접적인 부모로부터의 상속을 가진다면,
all of them will be analyzed on their own, even though only the most derived will be accessed directly on the blockchain. 이는 SMTChecker와 솔버에 불필요한 부담을 줍니다. 이러한 경우를 지원하기 위해
사용자는 배포된 컨트랙트로 분석해야 하는 컨트랙트를 지정할 수 있습니다. The parent contracts are of course still analyzed, but only in the context of the most derived contract, reducing the complexity of the encoding and generated queries. Note that abstract contracts are by default
not analyzed as the most derived by the SMTChecker.

선택된 컨트랙트들은 <source>:<contract>의 쌍으로 CLI를 통해 쉼표로 구분된 리스트를 통해 제공될 수 있습니다:
``--model-checker-contracts "<source1.sol:contract1>,<source2.sol:contract2>,<source2.sol:contract3>"``,
그리고 다음 형식을 갖는 :ref:`JSON input<compiler-api>`의 객체 ``settings.modelChecker.contracts``를 통해 제공될 수 있습니다:


.. code-block:: json

    "contracts": {
        "source1.sol": ["contract1"],
        "source2.sol": ["contract2", "contract3"]
    }

Reported Inferred Inductive Invariants
======================================

CHC 엔진으로 안전하다고 입증된 프로퍼티의 경우,
SMTChecker는 Horn 솔버가 이에 대한 증명의 일부로 추론했던 귀납적 불변성을 검색할 수 있습니다.
현재 두 가지 유형의 불변성이 사용자에게 보고될 수 있습니다:

- 컨트랙트 불변성: 이는 컨트랙트가 실행할 수 있는 모든 가능한 트랜잭션 전후에 참인 컨트랙트의
  상태 변수에 대한 프로퍼티입니다. 예를 들어, ``x >= y``에서 ``x``와 ``y``는 컨트랙트의 상태 변수입니다.
- 재진입 프로퍼티 (Reentrancy Properties): 이는 알 수 없는 코드에 대한 외부 호출이 있는 경우의
  컨트랙트의 동작, 행위를 나타냅니다. 이러한 프로퍼티는 외부 호출 전후의 상태 변수 값 사이의 관계를
  표현할 수 있습니다. 여기서 외부 호출은 분석된 컨트랙트에 대한 재진입 호출을 포함하여 어떤 것이든
  될 수 있습니다. 준비된 변수는 외부 호출 이후 상태 변수의 값을 나타냅니다. 예를 들면 다음과 같습니다: ``lock -> x = x'``.

사용자는 ``--model-checker-invariants "contract,reentrancy"`` 와 같은 CLI 옵션을 사용하거나 ``settings.modelChecker.invariants``라고 하는 :ref:`JSON input<compiler-api>`에서 제공되는 배열 필드를 활용하여 보고할 불변성에 대한 선택을 할 수 있습니다. 기본적으로 SMTChecker는 불변성에 대한 보고를 수행하지 않습니다.

슬랙 변수들과의 나눗셈이나 모듈로 연산(Division and Modulo With Slack Variables)
========================================

SMTChecker에서 사용하는 기본 Horn 솔버인 Spacer는 종종 Horn 규칙 내에서 나눗셈 및 모듈로
연산을 선호하지 않습니다. 이 때문에 기본적으로 Solidity의 나눗셈과 모듈로 연산은 ``a = b * d + m``
제약 조건을 사용하여 인코딩됩니다. 여기서 ``d = a / b``이며 ``m = a % b`` 입니다.
하지만, Eldarica 등과 같은 다른 솔버들은 구문상 정확한 연산을 선호합니다.
커맨드라인 플래그인 ``--model-checker-div-mod-no-slacks``와 JSON 옵션인 ``settings.modelChecker.divModNoSlacks``는 사용된 솔버 기본 설정에 따라 인코딩을 전환하는 데
사용할 수 있습니다.

Natspec 함수 추상화(Natspec Function Abstraction)
============================

``pow``나 ``sqrt``와 같은 흔한 수학적 메서드들을 포함한 특정한 함수들은 완전히 자동화된 방법으로
분석하기에 매우 복잡할 수 있습니다. 이러한 함수들은 Natspec 태그를 이용해 표현될 수 있습니다.
Netspec 태그는 SMTChecker에게 이 함수들은 추상화되는 것이 좋다고 암시해줍니다. 이것이 의미하는 바는
함수들의 몸체는 사용되지 않고 호출이 되었을 때, 함수들이 다음과 같은 행위를 할 것임을 의미합니다:

- 비결정적 값을 반환하고 추상화된 함수가 view/pure 함수 유형인 경우 상태 변수를 변경하지 않고
  유지하거나 그렇지 않으면 상태 변수를 비결정적인 값으로 설정합니다. 이 기능은 해당 어노테이션을 통해
  사용될 수 있습니다, ``/// @custom:smtchecker abstract-function-nondet``.
- 해석되지 않은 함수로 작동합니다. 이것이 의미하는 바는 함수의 (본체에 의해 제공된) 
  시맨틱들이 무시된다는 것을 의미합니다. 그리고 이 함수가 가진 유일한 속성은 동일한 입력이 주어지면
  동일한 출력을 보장한다는 것입니다. 이는 현재 개발 중에 있으며 ``/// @custom:smtchecker abstract-function-uf`` 어노테이션을 통해 사용이 가능할 예정입니다.

.. _smtchecker_engines:

모델 검증 엔진 (Model Checking Engines)
======================

SMTChecker 모듈은 두 가지의 다른 추론 엔진을 구현합니다. 각각 Bounded Model Checker (BMC)와
Constrained Horn Cluases (CHC) 시스템입니다. 두 엔진은 현재 개발 중에 있습니다. 그리고
서로 다른 특징들을 가집니다. 그 엔진들은 독립적이며 모든 프로퍼티 경고는 그것이 나온 엔진을
나타냅니다. 반례가 있는 위의 모든 예시는 더 강력한 엔진인 CHC에서 보고한 것입니다.

기본적으로 두 엔진이 모두 사용되며 여기서 CHC가 먼저 실행되고 검증되지 않은 모든 프로퍼티는
BMC로 전달됩니다. 당신은 CLI 옵션인 ``--model-checker-engine {all,bmc,chc,none}`` 또는
JSON 옵션인 ``settings.modelChecker.engine={all,bmc,chc,none}``를 통해서 특정한 엔진을
선택할 수 있습니다.

Bounded Model Checker (BMC)
---------------------------

BMC 엔진은 격리 구역의 함수들을 분석합니다. 즉, 각 기능을 분석할 때 여러 트랜잭션에 대한
계약의 전체 동작을 고려하지 않음을 의미합니다. 루프들은 또한 이 엔진에서 현재 무시됩니다.
내부 함수 호출들은 직간접적으로 재귀적이지 않은 한 인라인화됩니다. 외부 함수 호출들은
가능하다면 인라인화 됩니다. Knowledge that is potentially affected by reentrancy is erased.

위 특징들로 인해 BMC는 오탐을 보고하는 경향이 있지만, 가볍고 작은 로컬 버그들을
빠르게 찾을 수 있어야 합니다.

Constrained Horn Clauses (CHC)
------------------------------

A contract's Control Flow Graph (CFG) is modelled as a system of
Horn clauses, where the life cycle of the contract is represented by a loop
that can visit every public/external function non-deterministically. This way,
the behavior of the entire contract over an unbounded number of transactions
is taken into account when analyzing any function. Loops are fully supported
by this engine. Internal function calls are supported, and external function
calls assume the called code is unknown and can do anything.

The CHC engine is much more powerful than BMC in terms of what it can prove,
and might require more computing resources.

SMT 그리고 Horn 솔버 (SMT and Horn solvers)
====================

위에서 기술된 두 가지의 엔진은 논리적 백엔드로 자동화된 theorem prover를 사용합니다.
BMC는 SMT solver를 사용하고 CHC는 Horn solver를 사용합니다. 종종 동일한 도구가 
SMT 솔버의 역할을 하는 `z3 <https://github.com/Z3Provder/z3>`_ 와 Horn 솔버로
사용 가능한 `Spacer <https://spacer.bitbucket.io/>`_, 그리고 두 가지를 모두 수행하는
`Eldarica <https://github.com/uuverifiers/eldarica>`_ 의 역할 모두를 수행할 수 있습니다.

사용자는 CLI 옵션인 ``--model-checker-solvers {all,cvc4,smtlib2,z3}`` 옵션이나 
JSON 옵션인 ``settings.modelChecker.solvers=[smtlib2,z3]`` 옵션을 통해,
어떤 솔버가 사용될 것인지를 선택할 수 있습니다:

- ``cvc4`` 는 오직 ``solc`` 바이너리가 컴파일된 경우에만 사용할 수 있습니다. 오직 BMC만이 ``cvc4``를 사용합니다.
- ``smtlib2``는 SMT/Horn 쿼리를 `smtlib2 <http://smtlib.cs.uiowa.edu/>`_ 포맷으로 출력합니다.
  이는 컴파일러의 `callback mechanism <https://github.com/ethereum/solc-js>`_와 함께 사용될 수 
  있습니다. 따라서 시스템의 모든 솔버 바이너리를 사용해 쿼리 결과를 컴파일러에 동기적으로 반환할 수
  있습니다.
  이것은 C++ API가 없기 때문에 현재는 Eldarica를 사용하는 것이 유일한 방법입니다.
  이것은 호출되는 솔버에 따라 BMC와 CHC 모두에서 사용할 수 있습니다.
- ``z3``는 사용 가능합니다.

  - 만약 ``solc``가 다음과 함께 컴파일되었다면;
  - 만약 버전 4.8.x의 동적 ``z3`` 라이브러리가 리눅스 시스템(Solidity가 0.7.6 버전인)에 설치되어 있다면;
  - 정적으로 ``soljson.js`` (Solidity 0.6.9 버전 부터)안에 있습니다, 즉 컴파일러의 Javascript
    바이너리에 있습니다.

BMC와 CHC 모두 ``z3``를 사용하고 ``z3``는 브라우저를 포함하여 더 다양한 환경에서 사용할 수 있으므로
대부분의 사용자는 이 옵션에 대해 거의 걱정할 필요가 없습니다. 숙련된 사용자는 이 옵션을 적용하여
더 복잡한 문제에 대한 대체 솔버로 사용을 시도할 수 있습니다.

선택한 엔진과 솔버의 특정 조합은 예를 들면 CHC 및 ``cvc4``를 선택하는 것과 같이 SMTChecker가
아무 작업도 수행하지 않도록 합니다.

*******************************
Abstraction and False Positives
*******************************

SMTChecker는 불완전하고 건전한 방법으로 추상화를 구현합니다: 만약 버그가 리포트되었다면,
추상화에 의해 도입된 False positive일 가능성이 있습니다. (due to
erasing knowledge or using a non-precise type). 만약 검증 대상이 안전하다고 판단되면 실제로
안전합니다. 즉, false negative가 없습니다. (SMTChecker에 버그가 없는 한에는 그렇습니다.)

만약 해당 타겟을 증명할 수 없는 경우 이전 섹션의 조정 가능한 옵션을 사용하여 솔버를 도울 수 있습니다.
만약 당신이 false positive라는 것을 확신한다면, ``require`` 상태구문을 더 많은 정보와 함께 
코드에 추가함으로 솔버에 대해 더 많은 힘을 부여할 수도 있습니다.

SMT 인코딩과 타입 (SMT Encoding and Types)
======================

SMTChecker 인코딩은 아래 표와 같이 Solidity 유형과 표현식을 가장 가까운
`SMT-LIB <http://smtlib.cs.uiowa.edu/>`_ 표현 방식으로 매핑하여 가능한 한 정확하게 표현하려고
합니다.

+-----------------------+--------------------------------+-----------------------------+
|Solidity type          |SMT sort                        |Theories                     |
+=======================+================================+=============================+
|Boolean                |Bool                            |Bool                         |
+-----------------------+--------------------------------+-----------------------------+
|intN, uintN, address,  |Integer                         |LIA, NIA                     |
|bytesN, enum, contract |                                |                             |
+-----------------------+--------------------------------+-----------------------------+
|array, mapping, bytes, |Tuple                           |Datatypes, Arrays, LIA       |
|string                 |(Array elements, Integer length)|                             |
+-----------------------+--------------------------------+-----------------------------+
|struct                 |Tuple                           |Datatypes                    |
+-----------------------+--------------------------------+-----------------------------+
|other types            |Integer                         |LIA                          |
+-----------------------+--------------------------------+-----------------------------+

아직 지원되지 않는 유형은 지원되지 않는 작업이 무시되는 단일 256비트 부호 없는 정수로
추상화됩니다.

SMT 인코딩이 내부적으로 어떻게 동작하는지에 대해 추가적으로 알고 싶다면, 다음 문서를 확인하세요
`SMT-based Verification of Solidity Smart Contracts <https://github.com/leonardoalt/text/blob/master/solidity_isola_2018/main.pdf>`_.

함수 호출 (Function Calls)
==============

BMC 엔진에서, 같은 컨트랙트 (혹은 기본 컨트랙트) 에 대한 함수 호출은 가능하다면 인라인화됩니다,
즉, 구현이 가능할 때 인라인화됩니다. 다른 컨트랙트의 함수에 대한 호출은 코드가 사용 가능하더라도
인라인화되지 않습니다. 실제 배포된 코드가 동일하다고 보장할 수 없기 때문입니다.

CHC 엔진은 호출된 함수의 요약을 사용하여 내부 함수 호출을 지원하는 비선형 Horn 절을 생성합니다.
외부 함수 호출은 잠재적인 재진입 호출을 포함하여 알려지지 않은 코드 호출로 간주됩니다.

복잡한 퓨어 함수의 경우 안수에 대해 해석되지 않은 함수(Uninterpreted function:UF)에 의해 추상화됩니다.

+-----------------------------------+--------------------------------------+
|Functions                          |BMC/CHC behavior                      |
+===================================+======================================+
|``assert``                         |Verification target.                  |
+-----------------------------------+--------------------------------------+
|``require``                        |Assumption.                           |
+-----------------------------------+--------------------------------------+
|internal call                      |BMC: Inline function call.            |
|                                   |CHC: Function summaries.              |
+-----------------------------------+--------------------------------------+
|external call to known code        |BMC: Inline function call or          |
|                                   |erase knowledge about state variables |
|                                   |and local storage references.         |
|                                   |CHC: Assume called code is unknown.   |
|                                   |Try to infer invariants that hold     |
|                                   |after the call returns.               |
+-----------------------------------+--------------------------------------+
|Storage array push/pop             |Supported precisely.                  |
|                                   |Checks whether it is popping an       |
|                                   |empty array.                          |
+-----------------------------------+--------------------------------------+
|ABI functions                      |Abstracted with UF.                   |
+-----------------------------------+--------------------------------------+
|``addmod``, ``mulmod``             |Supported precisely.                  |
+-----------------------------------+--------------------------------------+
|``gasleft``, ``blockhash``,        |Abstracted with UF.                   |
|``keccak256``, ``ecrecover``       |                                      |
|``ripemd160``                      |                                      |
+-----------------------------------+--------------------------------------+
|pure functions without             |Abstracted with UF                    |
|implementation (external or        |                                      |
|complex)                           |                                      |
+-----------------------------------+--------------------------------------+
|external functions without         |BMC: Erase state knowledge and assume |
|implementation                     |result is nondeterminisc.             |
|                                   |CHC: Nondeterministic summary.        |
|                                   |Try to infer invariants that hold     |
|                                   |after the call returns.               |
+-----------------------------------+--------------------------------------+
|transfer                           |BMC: Checks whether the contract's    |
|                                   |balance is sufficient.                |
|                                   |CHC: does not yet perform the check.  |
+-----------------------------------+--------------------------------------+
|others                             |Currently unsupported                 |
+-----------------------------------+--------------------------------------+

추상화를 사용한다는 것은 정확한 정보의 상실을 의미하지만, 많은 경우 증명 수준, 증명력의
상실을 의미하지는 않습니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    contract Recover
    {
        function f(
            bytes32 hash,
            uint8 _v1, uint8 _v2,
            bytes32 _r1, bytes32 _r2,
            bytes32 _s1, bytes32 _s2
        ) public pure returns (address) {
            address a1 = ecrecover(hash, _v1, _r1, _s1);
            require(_v1 == _v2);
            require(_r1 == _r2);
            require(_s1 == _s2);
            address a2 = ecrecover(hash, _v2, _r2, _s2);
            assert(a1 == a2);
            return a1;
        }
    }

위에 제시된 예시에서, SMTChecker는 실제로 ``ecrecover``를 계산할 만큼의 표현력이 부족하지만,
함수 호출을 해석되지 않은 함수로 모델링함으로써 동등한 매개변수에서 호출될 때 반환 값이
동일하다는 것을 알 수 있습니다. 이것은 위에 제시된 assertion이 항상 참이라는 것을 증명하기에는
충분합니다.

UF와 함께 함수 호출을 추상화하는 것은 결정적이라고 알려진 함수에 대해 수행할 수 있으며
순수 함수에 대해 쉽게 수행할 수 있습니다. 그러나 상태 변수에 의존할 수 있기 때문에 일반적인
외부 함수로 이 작업을 수행하는 것은 어렵습니다.

Reference Types and Aliasing
============================

솔리디티는 동일한 :ref:`data location<data-location>`을 사용하여 레퍼런스 유형에 대한
앨리어싱을 구현합니다.
이것은 변수가 같은 데이터 영역에 있는 레퍼런스를 통해 수정이 될 수 있음을 의미합니다.
SMTChecker는 어떤 레퍼런스가 같은 데이터를 참조하는지에 대한 트랙을 유지하지 않습니다.
이는 지역 참조 또는 레퍼런스 유형의 상태 변수가 할당될 때마다 동일한 유형 및 데이터 위치의
변수에 대한 모든 정보가 지워진다는 것을 의미합니다.
만약 타입이 중첩되었다면, 정보를 제거하는데에는 모든 접두사 기본 유형 또한 포함됩니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0;

    contract Aliasing
    {
        uint[] array1;
        uint[][] array2;
        function f(
            uint[] memory a,
            uint[] memory b,
            uint[][] memory c,
            uint[] storage d
        ) internal {
            array1[0] = 42;
            a[0] = 2;
            c[0][0] = 2;
            b[0] = 1;
            // Erasing knowledge about memory references should not
            // erase knowledge about state variables.
            assert(array1[0] == 42);
            // However, an assignment to a storage reference will erase
            // storage knowledge accordingly.
            d[0] = 2;
            // Fails as false positive because of the assignment above.
            assert(array1[0] == 42);
            // Fails because `a == b` is possible.
            assert(a[0] == 2);
            // Fails because `c[i] == b` is possible.
            assert(c[0][0] == 2);
            assert(d[0] == 2);
            assert(b[0] == 1);
        }
        function g(
            uint[] memory a,
            uint[] memory b,
            uint[][] memory c,
            uint x
        ) public {
            f(a, b, c, array2[x]);
        }
    }

``b[0]``에 대한 대입 이후, 우리는 ``a``가 같은 타입 (``uint[]``) 와 데이터 위치(메모리)를
가지므로 이에 대한 정보를 초기화해야 합니다. 또한 기본 타입이 메모리에 있는 ``uint[]``이기 때문에
``c``에 대한 정보 또한 정리해야 합니다. 이것은 몇몇의 ``c[i]``가 ``b``또는 ``a``로 같은 데이터를
참조할 수 있다는 것을 암시합니다.

``array``와 ``d``에 대한 데이터는 스토리지에 있기 때문에 명확하지 않다는 점에 유의해야합니다.
이는 ``uint[]`` 유형도 포함하고 있기 때문입니다. 하지만, 만약 ``d``가 할당되었다면,
우리는 ``array``와 그 반대의 경우에 해당하는 데이터, 정보들을 정리해야 합니다.

Contract Balance
================

배포 트랜잭션에서 ``msg.value`` > 0인 경우 자금이 전송된 상태로 컨트랙트가 배포될 수 있습니다.
하지만, 컨트랙트의 주소에는 컨트랙트에 의해 유지되는 배포 전의 자금이 이미 있을 수 있습니다.
따라서, SMTChecker는 EVM 규칙과 일치하게 하기 위해 생성자에서 ``address(this).balance >= msg.value``를
가정합니다.
컨트랙트의 밸런스는 컨트랙트에 대한 어떠한 호출의 실행 없이 증가될 수 있습니다, 만약

- ``selfdestruct``는 분석된 컨트랙트를 나머지 자금의 대상으로 하는 다른 컨트랙트에 의해 실행되며,
- 컨트랙트는 몇몇 블록의 코인페이스 (i.e., ``block.coinbase``)입니다.

이를 적절하게 모델링하기 위한 SMTChecker는 모든 새로운 트랜잭션에서 컨트랙트의 잔액이 최소한
``msg.value``만큼 증가할 수 있다고 가정합니다.

**********************
Real World Assumptions
**********************

일부 시나리오는 Solidity 및 EVM으로 표현할 수 있지만 실제로는 발생하지 않을 것으로 예상됩니다.
이러한 경우 중 하나는 푸시 중에 오버플로되는 동적 스토리지 배열의 길이입니다. ``push`` 작업의 길이가
2^256 - 1인 배열에 적용되면 길이가 자동으로 오버플로됩니다.
하지만 배열을 해당 지점까지 확장하는 데 필요한 작업을 실행하는데 수십억 년이 걸리므로 실제로는
이러한 일이 일어나지 않을 것입니다.
SMTChecker가 취하는 또 다른 비슷한 가정은 주소의 잔액이 절대 오버플로될 수 없다는 것입니다.

비슷한 아이디어는 다음 문서에 제시되어 있습니다, `EIP-1985 <https://eips.ethereum.org/EIPS/eip-1985>`_.
