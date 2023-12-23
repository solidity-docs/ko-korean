********************************
솔리디티 소스파일의 레이아웃
********************************

소스파일은 임의 개의  :ref:`contract definitions<contract_structure>`, import_ 지시문과,
:ref:`pragma directives<pragma>` 그리고
:ref:`struct<structs>`, :ref:`enum<enums>`, :ref:`function<functions>`, :ref:`error<errors>`
:ref:`constant variable<constants>` 등의 정의들을 포함할 수 있습니다.

.. index:: ! license, spdx

SPDX 라이센스 식별자
=======================

Trust in smart contracts can be better established if their source code
is available. Since making source code available always touches on legal problems
with regards to copyright, the Solidity compiler encourages the use
of machine-readable `SPDX license identifiers <https://spdx.org>`_.
Every source file should start with a comment indicating its license:

스마트 컨트랙트에 대한 신뢰는 컨트랙트의 소스코드가 사용 가능한 상태일 때 더 잘 구축될 수 있습니다.
소스코드를 이용할 수 있게 하는 것은 저작권과 관련한 법적 문제를 항상 다루고 있으므로,
솔리디티 컴파일러는 machine-readable `SPDX license identifiers <https://spdx.org>`_의 이용을
장려하고 있습니다. 모든 소스파일은 해당 라이센스를 나타내는 주석으로 시작해야합니다.

``// SPDX-License-Identifier: MIT``

컴파일러는 라이센스가 `list allowed by SPDX <https://spdx.org/licenses/>`_의 일부인지는
확인하지 않지만, 제공된 문자열을 :ref:`bytecode metadata <metadata>`에 포함하고 있습니다.

만약 라이센스를 지정하지 않고 싶거나 소스코드가 오픈소스인 경우,
특별한 값인 ``UNLICENSED``를 사용해 주십시오.

물론 이 주석을 제공한다고 해도 각 소스파일 내 특정 라이센스 헤더를 언급해야 한다거나
원본 저작권자를 언급해야 하는 등 라이센싱과 관련된 다른 의무로부터 자유로워지는 것은 아닙니다.

이 주석은 어디에 있든지 컴파일러에 의해 인지되지만,
파일 맨 위에 적는 것을 추천합니다.

SPDX 라이센스 식별자에 대해 더 많은 정보를 찾고 싶으면,
`SPDX website <https://spdx.org/ids-how>`_에서 찾으실 수 있습니다.

.. index:: ! pragma

.. _pragma:

프래그마(pragma)
=======

``프래그마`` 키워드는 특정 컴파일러 기능 또는 검사를 활성하하는 데에 사용됩니다.
pragma 지시문은 항상 소스파일에만 종속되어서, 만약 모든 프로젝트의
모든 파일에 프래그마를 활성화하고 싶다면 이를 모든 파일에 추가해야합니다. 
.. index:: ! pragma, version

.. _version_pragma:

버전 프래그마
--------------

소스파일은 버전 프래그마를 주석으로 달아 호환되지 않는 변경사항을 도입할 지도 모르는
미래 컴파일러 버전과의 컴파일을 거부할 수 있습니다(해야만 합니다.).
우리는 이 변경사항을 절대적으로 최소화하고, 의미론의 변화 또는 구문의 변화를
요구하는 방식으로 도입하기 위해 노력하고 있습니다만, 이것이 항상
가능한 것은 아닙니다. 따라서 최소한의 변경 사항이 포함된 버전의 경우
변경로그를 통해 읽는 것이 항상 좋은 방법입니다. 이러한 버전은 항상 
``0.x.0`` 또는 ``x.0.0`` 형태를 가지고 있습니다.

버전 프래그마는 다음과 같이 사용됩니다. ``pragma solidity ^0.5.2;``

위 라인이 쓰여진 소스파일은 0.5.2 보다 이전 버전의 컴파일러로 컴파일되지 않고,
버전 0.6.0으로 시작하는 컴파일 위에서 동작하지 않습니다(두 번째 조건은 ``^``를 사용하여 추가됩니다.).
``0.6.0``버전까지는 변경사항이 없을 것이기 때문에, 코드가 의도한 대로 컴파일될 것임을 확신할 수 있습니다.
컴파일러의 정확한 버전은 고정되지 않아서, 버그 수정 릴리즈는 언제나 가능합니다.

컴파일러 버전에 대한 더 복잡한 규칙을 명시할 수 있고, 이는
`npm <https://docs.npmjs.com/cli/v6/using-npm/semver>`_를 사용한 것과 같은 구문을 따릅니다.

.. note::

  버전 프래그마를 사용하는 것은 컴파일러의 버전을 *바꾸는* 것이 아닙니다.
  또한 컴파일러의 특징을 *활성화/비활성화* 하는 것도 아닙니다.
  단지 프래그마에 의해 요구된 버전에 부합하는지 확인하기 위해 컴파일러에게 알려주는 것 뿐입니다.
  버전이 맞지 않다면, 컴파일러는 에러를 출력합니다.

ABI 코더 프래그마 
----------------

``pragma abicoder v1`` 와 ``pragma abicoder v2``를 이용함으로써,
ABI 인코더와 디코더 두 가지 구현 중 하나를 선택할 수 있습니다.

새로운 ABI 코더 (v2)는 임의적으로 둘러싸인 배열과 구조를 인코딩, 디코딩할 수 있습니다.
이는 최적이 아닌 코드를 생산할 지도 모르고 예전 인코더만큼 많은 테스트를 받지 못했을지
몰라도, 솔리디티 0.6.0이서는 비실험적인 것으로 여겨집니다.
여전히 ``pragma abicoder v2;``를 사용해 명시적으로 활성화할 수 있습니다.
솔리디티 0.8.0으로 시작하는 경우 기본적으로 활성화되기 떄문에, 
``pragma abicoder v1;``를 사용하여 옛 버전의 코더를 선택할 수 있는 옵션이 있습니다.

새로운 인코더가 지원하는 타입 집합은 기존 인코더가 지원하는 타입의 완전한
상위집합입니다. 이를 사용하는 컨트랙트는 이를 사용하지 않는 컨트랙트와 제약 없이
상호작용할 수 있습니다. 그 반대는 ``abicoder v2``가 아닌 컨트랙트가 
'새로운 인코더의 지원을 받는 디코딩 타입을 필요로 하는 호출'을 시도하지 
않을 때 가능합니다. 컴파일러는 이를 발견하고 에러를 출력할 수 있습니다.
컨트랙트를 위해 ``abicoder v2``를 활성화하는 것은 위의 에러를 없앨 수 있습니다.


.. note::

  이 프래그마는  코드가 결국 어디서 끝났는지는 상관 없이, 프래그마가 정의된 
  활성화된 파일 내 모든 코드에 적용됩니다. 이는 ABI 코더 v1으로 
  컴파일되도록 선택된 컨트랙트의 소스파일이 다른 컨트랙트에서부터 그것을 물려받음으로써
  여전히 새로운 인코더를 사용하는 코드를 포함할 수 있다는 것을 의미합니다.
  이것은 새로운 타입이 내부적으로만 사용되고, 외부 함수 기호로 사용되지 않았을 때
  허용됩니다. 


.. note::

  솔리디티 0.7.4까지는 ``pragma experimental ABIEncoderV2``를 사용함으로써
  ABI coder v2를 선택하는 것이 가능했지만, coder v1은 기본값이었기 때문에 명시적
  으로 선택하는 것이 불가능했습니다.

.. index:: ! pragma, experimental

.. _experimental_pragma:


실험적 프래그마
-------------------

두 번째 프래그마는 실험적 프래그마입니다. 컴파일러의 기능이나 아직 기본으로 
활성화되지 않은 언어를 활성화하는데에 사용될 수 있습니다.


ABIEncoderV2
~~~~~~~~~~~~

ABI 코더 v2는 더 이상 실험적으로 여겨지지 않아서, 솔리디티 0.7.4부터는
``pragma abicoder v2``프래그마를 이용하여 선택할 수 있게 되었습니다.(위를 참고하세요.)

.. _smt_checker:

SMTChecker
~~~~~~~~~~

이 컴포넌트는 솔리디티 컴파일러가 빌드되면 활성화되어야 하므로 모든 솔리디티 바이너리에서
사용가능한 것은 아닙니다. :ref:`build instructions<smt_solvers_build>`은 
이 옵션을 어떻게 활성화하는지 설명합니다. 대부분의 버전의 경우
우분투 PPA 릴리즈에 대해 활성화되지만, 도커 이미지, 윈도우 바이너리 또는
정적-구축 리눅스 바이너리에 대해서는 활성화되지 않습니다. solc-js에 대해서는 만약 로컬에 설치된 'SMT solver'를 가지고 있고 (브라우저를 통해서가 아닌) 
노드를 통해 solc-js를 실행하는 경우 `smtCallback <https://github.com/ethereum/solc-js#example-usage-with-smtsolver-callback>`_를 통해 활성화가 가능합니다.

만약 ``pragma experimental SMTChecker;``을 사용한다면, 'SMT solver'에 질의를 보냄으로써 생기는 추가적인 :ref:`safety warnings<formal_verification>`을 얻습니다.
이 컴포넌트는 아직 솔리디티 언어의 모든 기능을 지원하지 않고 많은 경고를 초래할 수 있습니다. 지원하지 않는 기능의 경우 분석이 정확하지 않을 수 있습니다.


.. index:: source file, ! import, module, source unit

.. _import:

다른 소스 파일 가져오기
============================

구문과 의미
--------------------

솔리디티는 코드를 자바스크립트에서 이용가능한 것과 비슷하게
모듈화하는 것을 돕기 위해 코드문을(ES6부터) 가져오는 것을 지원합니다.
그러나, 솔리디티는 `default export <https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Description>`_
의 컨셉은 지원하지 않습니다.

전역 수준에서, 다음과 같은 형태로 코드문을 가져올 수 있습니다.

.. code-block:: solidity 솔리디티

    import "filename";

``filename`` 부분을 *import path*라고 부릅니다.
이 구문은 "filename"의 모든 전역 심볼(과 그 곳에 불려온 심볼들)을 현재의 전역 영역으로
가져옵니다(ES6 과는 다르지만 Solidity의 하위호환입니다.).
이 형태는 예상치 못한 namespace 오염을 유발할 수 있기 때문에 사용에는 추천되지 않습니다. 
만약 새로운 top-level 아이템들을 "filename"에 추가하였다면, 모든 파일에 "filename" 형태와 같이 가져온 것들이 자동으로 나타납니다.
특정 심볼을 명시적으로 가져오는 것이 낫습니다.

다음 예제는 ``"filename"``으로부터 온 전역 심볼을 멤버로 가지는 새로운 전역 심볼 ``symbolName``을 생성합니다.

.. code-block:: solidity

    import * as symbolName from "filename";

이는 모든 전역 심볼을 ``symbolName.symbol`` 형태로 사용할 수 있게 해 줍니다.

ES6의 일부는 아니지만 유용할 수 있는 이 구문의 변형은 다음과 같습니다.

.. code-block:: solidity

  import "filename" as symbolName;

이는 ``import * as symbolName from "filename";``과 같습니다.

이름 중복이 있다면, import 중에 심볼 이름을 바꿀 수 있습니다. 예를 들어.
아래 코드가 새로운 전역 심볼 ``alias``와 ``symbol2``을 만들어내는데 이는 각각 ``"filename"``
의 내부에서 ``symbol1``와 ``symbol2``를 지칭합니다.

.. code-block:: solidity

    import {symbol1 as alias, symbol2} from "filename";

.. index:: virtual filesystem, source unit name, import; path, filesystem path, import callback, Remix IDE

경로(Paths) 가져오기
------------

솔리디티 컴파일러가 모든 플랫폼에서 재현 가능한 빌드를 지원려면 소스 파일이 저장된
파일 시스템의 세부 정보를 추상화해야 합니다. 이러한 이유로 경로를 가져오는 것은 호스트
파일 시스템의 파일을 직접적으로 참조하지 않습니다. 대신 컴파일러는 
각 소스 단위에 불투명하고 비구조화된 식별자인 고유한 *소스 단위 이름* 할당되는 내부 데이터베이스
(*virtual filesystem* 또는 약자로 *VFS*)를 유지합니다.

:ref:`Standard JSON <compiler-api>` API를 사용함으로써 모든 소스파일의 이름과
내용을 컴파일러 입력값의 일부로써 직접적으로 제공하는 것이 가능합니다.
위의 경우 소스 단위 이름은 정말로 임의적입니다.
그러나 만약 컴파일러가 자동적으로 소스를 찾고 VFS에 로드하는 것을 원한다면, 소스 단위 이름을
:ref:`import callback<import-callback>`에서 찾을 수 있도록 구성해야 합니다.
커맨드라인 컴파일러를 사용할 때 기본 가져오기 콜백은 호스트 파일시스템에서 소스코드를 로드하는
기능만 지원하므로, 이는 소스 유닛 이름이 paths이어야 함을 의미합니다.
몇몇 환경은 보다 다양한 사용자 정의 콜백을 제공합니다. 예를 들어, `Remix IDE 
<https://remix.ethereum.org/>`_는 `HTTP, IPFS 및 Swarm URL에서 파일을 가져오거나 NPM 레지스트리의 패키지를 직접 참조할 수 있습니다
<https://remix-ide.readthedocs.io/en/latest/import.html>`_.

컴파일러가 사용하는 가상 파일 시스템과 path resolution logic의 완벽한 정의를 알고 싶으면
:ref:`Path Resolution <path-resolution>`를 참고하세요.

.. index:: ! comment, natspec

주석
========

한줄 주석 (``//``)과 여러줄 주석 (``/*...*/``)이 사용가능합니다.

.. code-block:: solidity

    // 이것은 한줄 주석입니다.

    /*
    이것은 여러줄
    주석입니다.
    */

.. note::

  한줄 주석은 UTF-8 인코딩의 유니코드 라인 터미네이터(LF, VF, FF, CR, NEL, LS or PS)
  에 의해 종료됩니다. 터미네이터는 주석 이후에 여전히 소스 코드의 한 부분이어서,
  ASCII 기호가 아닌 경우(NEL, LS, PS) 파서 오류가 발생합니다.


추가적으로 NatSpec comment 라고 불리는 다른 종류의 주석이 있는데, 이 주석은
:ref:`style guide<style_guide_natspec>`에 자세히 나와 있습니다. 이 주석은 
triple slash (``///``) 또는 double asterisk block (``/** ... */``)으로 작성되며
함수 선언이나 문장 바로 위에 사용해야 합니다.