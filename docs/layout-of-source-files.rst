********************************
Layout of a Solidity Source File
솔리디티 소스파일의 레이아웃
********************************

Source files can contain an arbitrary number of
:ref:`contract definitions<contract_structure>`, import_ directives,
:ref:`pragma directives<pragma>` and
:ref:`struct<structs>`, :ref:`enum<enums>`, :ref:`function<functions>`, :ref:`error<errors>`
and :ref:`constant variable<constants>` definitions.

소스파일은 임의 개의  :ref:`contract definitions<contract_structure>`, import_ 지시문과,
:ref:`pragma directives<pragma>` 그리고
:ref:`struct<structs>`, :ref:`enum<enums>`, :ref:`function<functions>`, :ref:`error<errors>`
:ref:`constant variable<constants>` 등의 정의들을 포함할 수 있습니다.

.. index:: ! license, spdx

SPDX License Identifier
SPDX License 식별자
=======================

Trust in smart contracts can be better established if their source code
is available. Since making source code available always touches on legal problems
with regards to copyright, the Solidity compiler encourages the use
of machine-readable `SPDX license identifiers <https://spdx.org>`_.
Every source file should start with a comment indicating its license:

스마트 컨트랙트에 대한 신뢰는 소스코드가 사용 가능할 때 더 잘 구축될 수 있습니다.
소스코드를 이용할 수 있게 하는 것은 저작권과 관련한 법적 문제를 항상 다루고 있으므로,
솔리디티 컴파일러는 machine-readable `SPDX license identifiers <https://spdx.org>`_의 이용을
장려하고 있습니다. 모든 소스파일은 해당 라이센스를 나타내는 주석으로 시작해야합니다.

``// SPDX-License-Identifier: MIT``

The compiler does not validate that the license is part of the
`list allowed by SPDX <https://spdx.org/licenses/>`_, but
it does include the supplied string in the :ref:`bytecode metadata <metadata>`.

컴파일러는 라이센스가 `list allowed by SPDX <https://spdx.org/licenses/>`_의 일부인지는
확인하지 않지만, 제공된 문자열을 :ref:`bytecode metadata <metadata>`에 포함하고 있습니다.

If you do not want to specify a license or if the source code is
not open-source, please use the special value ``UNLICENSED``.

만약 라이센스를 지정하지 않고 싶거나 소스코드가 오픈소스인 경우,
특별한 값인 ``UNLICENSED``를 사용해 주십시오.

Supplying this comment of course does not free you from other
obligations related to licensing like having to mention
a specific license header in each source file or the
original copyright holder.

물론 이 주석을 제공한다고 해도 각 소스파일 내 특정 라이센스 헤더를 언급해야 한다거나
원본 저작권자를 언급해야 하는 등 라이센싱과 관련된 다른 의무로부터 자유로워지는 것은 아닙니다.

The comment is recognized by the compiler anywhere in the file at the
file level, but it is recommended to put it at the top of the file.

이 주석은 어디에 있든지 컴파일러에 의해 인지되지만,
파일 맨 위에 적는 것을 추천합니다.

More information about how to use SPDX license identifiers
can be found at the `SPDX website <https://spdx.org/ids-how>`_.

SPDX license identifiers에 대해 더 많은 정보를 찾고 싶으면,
`SPDX website <https://spdx.org/ids-how>`_에서 찾으실 수 있습니다.

.. index:: ! pragma

.. _pragma:

Pragmas
=======

The ``pragma`` keyword is used to enable certain compiler features
or checks. A pragma directive is always local to a source file, so
you have to add the pragma to all your files if you want to enable it
in your whole project. If you :ref:`import<import>` another file, the pragma
from that file does *not* automatically apply to the importing file.

``프래그마`` 키워드는 특정 컴파일러 기능 또는 검사를 활성하하는 데에 사용됩니다.
pragma 지시문은 항상 소스파일에만 종속되어서, 만약 모든 프로젝트의
모든 파일에 pragma를 활성화하고 싶다면 이를 모든 파일에 추가해야합니다. 
.. index:: ! pragma, version

.. _version_pragma:

Version Pragma
버전 프래그마
--------------

Source files can (and should) be annotated with a version pragma to reject
compilation with future compiler versions that might introduce incompatible
changes. We try to keep these to an absolute minimum and
introduce them in a way that changes in semantics also require changes
in the syntax, but this is not always possible. Because of this, it is always
a good idea to read through the changelog at least for releases that contain
breaking changes. These releases always have versions of the form
``0.x.0`` or ``x.0.0``.

소스파일은 버전 프래그마를 주석으로 달아 호환되지 않는 변경사항을 도입할 지도 모르는
미래 컴파일러 버전과의 컴파일을 거부할 수 있습니다(해야만 합니다.).
우리는 이 변경사항을 절대적으로 최소화하고, 의미론의 변화 또는 구문의 변화를
요구하는 방식으로 도입하기 위해 노력하고 있습니다만, 이것이 항상
가능한 것은 아닙니다. 따라서 최소한의 변경 사항이 포함된 버전의 경우
변경로그를 통해 읽는 것이 항상 좋은 방법입니다. 이러한 버전은 항상 
``0.x.0`` 또는 ``x.0.0`` 형태를 가지고 있습니다.

The version pragma is used as follows: ``pragma solidity ^0.5.2;``
버전 프래그마는 다음과 같이 사용됩니다. ``pragma solidity ^0.5.2;``

A source file with the line above does not compile with a compiler earlier than version 0.5.2,
and it also does not work on a compiler starting from version 0.6.0 (this
second condition is added by using ``^``). Because
there will be no breaking changes until version ``0.6.0``, you can
be sure that your code compiles the way you intended. The exact version of the
compiler is not fixed, so that bugfix releases are still possible.

위 라인이 쓰여진 소스파일은 0.5.2 보다 이전 버전의 컴파일러로 컴파일되지 않고,
버전 0.6.0으로 시작하는 컴파일 위에서 동작하지 않습니다(두 번째 조건은 ``^``를 사용하여 추가됩니다.).
``0.6.0``버전까지는 변경사항이 없을 것이기 때문에, 코드가 의도한 대로 컴파일될 것임을 확신할 수 있습니다.
컴파일러의 정확한 버전은 고정되지 않아서, 버그 수정 릴리즈는 언제나 가능합니다.

It is possible to specify more complex rules for the compiler version,
these follow the same syntax used by `npm <https://docs.npmjs.com/cli/v6/using-npm/semver>`_.

컴파일러 버전에 대한 더 복잡한 규칙을 명시할 수 있고, 이는
`npm <https://docs.npmjs.com/cli/v6/using-npm/semver>`_를 사용한 것과 같은 구문을 따릅니다.

.. note::
  Using the version pragma *does not* change the version of the compiler.
  It also *does not* enable or disable features of the compiler. It just
  instructs the compiler to check whether its version matches the one
  required by the pragma. If it does not match, the compiler issues
  an error.

  버전 프래그마를 사용하는 것은 컴파일러의 버전을 *바꾸는* 것이 아닙니다.
  또한 컴파일러의 특징을 *활성화/비활성화* 하는 것도 아닙니다.
  단지 프래그마에 의해 요구된 버전에 부합하는지 확인하기 위해 컴파일러에게 알려주는 것 뿐입니다.
  버전이 맞지 않다면, 컴파일러는 에러를 출력합니다.

ABI Coder Pragma
ABI 코더 프래그마 
----------------

By using ``pragma abicoder v1`` or ``pragma abicoder v2`` you can
select between the two implementations of the ABI encoder and decoder.

``pragma abicoder v1`` 와 ``pragma abicoder v2``를 이용함으로써,
ABI 인코더와 디코더 두 가지 구현 중 하나를 선택할 수 있습니다.

The new ABI coder (v2) is able to encode and decode arbitrarily nested
arrays and structs. It might produce less optimal code and has not
received as much testing as the old encoder, but is considered
non-experimental as of Solidity 0.6.0. You still have to explicitly
activate it using ``pragma abicoder v2;``. Since it will be
activated by default starting from Solidity 0.8.0, there is the option to select
the old coder using ``pragma abicoder v1;``.

새로운 ABI 코더 (v2)는 임의적으로 둘러싸인 배열과 구조를 인코딩, 디코딩할 수 있습니다.
이는 최적이 아닌 코드를 생산할 지도 모르고 예전 인코더만큼 많은 테스트를 받지 못했을지
몰라도, 솔리디티 0.6.0이서는 비실험적인 것으로 여겨집니다.
여전히 ``pragma abicoder v2;``를 사용해 명시적으로 활성화할 수 있습니다.
솔리디티 0.8.0으로 시작하는 경우 기본적으로 활성화되기 떄문에, 
``pragma abicoder v1;``를 사용하여 옛 버전의 코더를 선택할 수 있는 옵션이 있습니다.

The set of types supported by the new encoder is a strict superset of
the ones supported by the old one. Contracts that use it can interact with ones
that do not without limitations. The reverse is possible only as long as the
non-``abicoder v2`` contract does not try to make calls that would require
decoding types only supported by the new encoder. The compiler can detect this
and will issue an error. Simply enabling ``abicoder v2`` for your contract is
enough to make the error go away.

새로운 인코더가 지원하는 타입 집합은 기존 인코더가 지원하는 타입의 완전한
상위집합입니다. 이를 사용하는 컨트랙트는 이를 사용하지 않는 컨트랙트와 제약 없이
상호작용할 수 있습니다. 그 반대는 ``abicoder v2``가 아닌 컨트랙트가 
새로운 인코더의 지원을 받는 디코딩 타입을 필요로 하는 호출을 시도하지 
않을 때 가능합니다. 컴파일러는 이를 발견하고 에러를 출력할 수 있습니다.
컨트랙트를 위해 ``abicoder v2``를 활성화하는 것은 위의 에러를 없앨 수 있습니다.


.. note::
  This pragma applies to all the code defined in the file where it is activated,
  regardless of where that code ends up eventually. This means that a contract
  whose source file is selected to compile with ABI coder v1
  can still contain code that uses the new encoder
  by inheriting it from another contract. This is allowed if the new types are only
  used internally and not in external function signatures.

  이 프래그마는  코드가 결국 어디서 끝났는지는 상관 없이, 프래그마가 정의된 
  활성화된 파일 내 모든 코드에 적용됩니다. 이는 ABI 코더 v1으로 
  컴파일되도록 선택된 컨트랙트의 소스파일이 다른 컨트랙트에서부터 그것을 물려받음으로써
  여전히 새로운 인코더를 사용하는 코드를 포함할 수 있다는 것을 의미합니다.
  이것은 새로운 타입이 내부적으로만 사용되고, 외부 함수 기호로 사용되지 않았을 때
  허용됩니다. 


.. note::
  Up to Solidity 0.7.4, it was possible to select the ABI coder v2
  by using ``pragma experimental ABIEncoderV2``, but it was not possible
  to explicitly select coder v1 because it was the default.

  솔리디티 0.7.4까지는 ``pragma experimental ABIEncoderV2``를 사용함으로써
  ABI coder v2를 선택하는 것이 가능했지만, coder v1은 기본값이었기 때문에 명시적
  으로 선택하는 것이 불가능했습니다.

.. index:: ! pragma, experimental

.. _experimental_pragma:

Experimental Pragma
실험적 프래그마
-------------------

The second pragma is the experimental pragma. It can be used to enable
features of the compiler or language that are not yet enabled by default.
The following experimental pragmas are currently supported:

두 번째 프래그마는 실험적 프래그마입니다. 컴파일러의 기능이나 아직 기본으로 
활성화되지 않은 언어를 활성화하는데에 사용될 수 있습니다.


ABIEncoderV2
ABIEncoderV2
~~~~~~~~~~~~

Because the ABI coder v2 is not considered experimental anymore,
it can be selected via ``pragma abicoder v2`` (please see above)
since Solidity 0.7.4.

ABI 코더 v2는 더 이상 실험적으로 여겨지지 않아서, 솔리디티 0.7.4부터는
``pragma abicoder v2``프래그마를 이용하여 선택할 수 있게 되었습니다.(위를 참고하세요.)

.. _smt_checker:

SMTChecker
SMTChecker
~~~~~~~~~~

This component has to be enabled when the Solidity compiler is built
and therefore it is not available in all Solidity binaries.
The :ref:`build instructions<smt_solvers_build>` explain how to activate this option.
It is activated for the Ubuntu PPA releases in most versions,
but not for the Docker images, Windows binaries or the
statically-built Linux binaries. It can be activated for solc-js via the
`smtCallback <https://github.com/ethereum/solc-js#example-usage-with-smtsolver-callback>`_ if you have an SMT solver
installed locally and run solc-js via node (not via the browser).

이 컴포넌트는 솔리디티 컴파일러가 빌드되면 활성화되어야 하므로 모든 솔리디티 바이너리에서
사용가능한 것은 아닙니다. :ref:`build instructions<smt_solvers_build>`은 
이 옵션을 어떻게 활성화하는지 설명합니다. 대부분의 버전의 경우
우분투 PPA 릴리즈에 대해 활성화되지만, Docker 이미지, 윈도우 바이너리 또는
정적-구축 리눅스 바이너리에 대해서는 활성화되지 않습니다. solc-js에 대해서는 만약 로컬에 설치된 SMT solver를 가지고 있고 (브라우저를 통해서가 아닌) 
노드를 통해 solc-js를 실행하는 경우 `smtCallback <https://github.com/ethereum/solc-js#example-usage-with-smtsolver-callback>`_를 통해 활성화가 가능합니다.


If you use ``pragma experimental SMTChecker;``, then you get additional
:ref:`safety warnings<formal_verification>` which are obtained by querying an
SMT solver.
The component does not yet support all features of the Solidity language and
likely outputs many warnings. In case it reports unsupported features, the
analysis may not be fully sound.

만약 ``pragma experimental SMTChecker;``을 사용한다면, SMT solver에 질의를 보냄으로써 생기는 추가적인 :ref:`safety warnings<formal_verification>`을 얻습니다.
이 컴포넌트는 아직 솔리디티 언어의 모든 기능을 지원하지 않고 많은 경고를 초래할 수 있습니다. 지원하지 않는 기능의 경우 분석이 정확하지 않을 수 있습니다.


.. index:: source file, ! import, module, source unit

.. _import:

Importing other Source Files
다른 소스 파일 가져오기
============================

Syntax and Semantics
구문과 의미
--------------------

Solidity supports import statements to help modularise your code that
are similar to those available in JavaScript
(from ES6 on). However, Solidity does not support the concept of
a `default export <https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Description>`_.

솔리디티는 코드를 자바스크립트에서 이용가능한 것과 비슷하게
모듈화하는 것을 돕기 위해 코드문을(ES6부터) 가져오는 것을 지원합니다.
그러나, 솔리디티는 `default export <https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Description>`_
의 컨셉은 지원하지 않습니다.

At a global level, you can use import statements of the following form:

전역 수준에서, 다음과 같은 형태로 코드문을 가져올 수 있습니다.

.. code-block:: solidity 솔리디티

    import "filename";

The ``filename`` part is called an *import path*.
This statement imports all global symbols from "filename" (and symbols imported there) into the
current global scope (different than in ES6 but backwards-compatible for Solidity).
This form is not recommended for use, because it unpredictably pollutes the namespace.
If you add new top-level items inside "filename", they automatically
appear in all files that import like this from "filename". It is better to import specific
symbols explicitly.

``filename`` 부분을 *import path*라고 부릅니다.
이 구문은 "filename"의 모든 전역 심볼(과 그 곳에 불려온 심볼들)을 현재의 전역 영역으로
가져옵니다(ES6 과는 다르지만 Solidity의 하위호환입니다.).
이 형태는 예상치 못한 namespace 오염을 유발할 수 있기 때문에 사용에는 추천되지 않습니다. 
만약 새로운 top-level 아이템들을 "filename"에 추가하였다면, 모든 파일에 "filename" 형태와 같이 가져온 것들이 자동으로 나타납니다.
특정 심볼을 명시적으로 가져오는 것이 낫습니다.

The following example creates a new global symbol ``symbolName`` whose members are all
the global symbols from ``"filename"``:

다음 예제는 ``"filename"``으로부터 온 전역 심볼을 멤버로 가지는 새로운 전역 심볼 ``symbolName``을 생성합니다.

.. code-block:: solidity

    import * as symbolName from "filename";

which results in all global symbols being available in the format ``symbolName.symbol``.

이는 모든 전역 심볼을 ``symbolName.symbol`` 형태로 사용할 수 있게 해 줍니다.

A variant of this syntax that is not part of ES6, but possibly useful is:

ES6의 일부는 아니지만 유용할 수 있는 이 구문의 변형은 다음과 같습니다.

.. code-block:: solidity

  import "filename" as symbolName;

which is equivalent to ``import * as symbolName from "filename";``.

이는 ``import * as symbolName from "filename";``과 같습니다.

If there is a naming collision, you can rename symbols while importing. For example,
the code below creates new global symbols ``alias`` and ``symbol2`` which reference
``symbol1`` and ``symbol2`` from inside ``"filename"``, respectively.

이름 중복이 있다면, import 중에 심볼 이름을 바꿀 수 있습니다. 예를 들어.
아래 코드가 새로운 전역 심볼 ``alias``와 ``symbol2``을 만들어내는데 이는 각각 ``"filename"``
의 내부에서 ``symbol1``와 ``symbol2``를 지칭합니다.

.. code-block:: solidity

    import {symbol1 as alias, symbol2} from "filename";

.. index:: virtual filesystem, source unit name, import; path, filesystem path, import callback, Remix IDE

Import Paths
Paths 가져오기
------------

In order to be able to support reproducible builds on all platforms, the Solidity compiler has to
abstract away the details of the filesystem where source files are stored.
For this reason import paths do not refer directly to files in the host filesystem.
Instead the compiler maintains an internal database (*virtual filesystem* or *VFS* for short) where
each source unit is assigned a unique *source unit name* which is an opaque and unstructured identifier.
The import path specified in an import statement is translated into a source unit name and used to
find the corresponding source unit in this database.

솔리디티 컴파일러가 모든 플랫폼에서 재현 가능한 빌드를 지원려면 소스 파일이 저장된
파일 시스템의 세부 정보를 추상화해야 합니다. 이러한 이유로 paths를 가져오는 것은 호스트
파일 시스템의 파일을 직접적으로 참조하지 않습니다. 대신 컴파일러는 
각 소스 단위에 불투명하고 비구조화된 식별자인 고유한 *소스 단위 이름* 할당되는 내부 데이터베이스
(*virtual filesystem* 또는 약자로 *VFS*)를 유지합니다.



Using the :ref:`Standard JSON <compiler-api>` API it is possible to directly provide the names and
content of all the source files as a part of the compiler input.
In this case source unit names are truly arbitrary.
If, however, you want the compiler to automatically find and load source code into the VFS, your
source unit names need to be structured in a way that makes it possible for an :ref:`import callback
<import-callback>` to locate them.
When using the command-line compiler the default import callback supports only loading source code
from the host filesystem, which means that your source unit names must be paths.
Some environments provide custom callbacks that are more versatile.
For example the `Remix IDE <https://remix.ethereum.org/>`_ provides one that
lets you `import files from HTTP, IPFS and Swarm URLs or refer directly to packages in NPM registry
<https://remix-ide.readthedocs.io/en/latest/import.html>`_.

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

For a complete description of the virtual filesystem and the path resolution logic used by the
compiler see :ref:`Path Resolution <path-resolution>`.

컴파일러가 사용하는 가상 파일 시스템과 path resolution logic의 완벽한 정의를 알고 싶으면
:ref:`Path Resolution <path-resolution>`를 참고하세요.

.. index:: ! comment, natspec

Comments
주석
========

Single-line comments (``//``) and multi-line comments (``/*...*/``) are possible.
한줄 주석 (``//``)과 여러줄 주석 (``/*...*/``)이 사용가능합니다.

.. code-block:: solidity

    // This is a single-line comment.
    // 이것은 한줄 주석입니다.

    /*
    This is a
    multi-line comment.
    */

    /*
    이것은 여러줄
    주석입니다.
    */

.. note::
  A single-line comment is terminated by any unicode line terminator
  (LF, VF, FF, CR, NEL, LS or PS) in UTF-8 encoding. The terminator is still part of
  the source code after the comment, so if it is not an ASCII symbol
  (these are NEL, LS and PS), it will lead to a parser error.

  한줄 주석은 UTF-8 인코딩의 유니코드 라인 터미네이터(LF, VF, FF, CR, NEL, LS or PS)
  에 의해 종료됩니다. 터미네이터는 주석 이후에 여전히 소스 코드의 한 부분이어서,
  ASCII 기호가 아닌 경우(NEL, LS, PS) 파서 오류가 발생합니다.

Additionally, there is another type of comment called a NatSpec comment,
which is detailed in the :ref:`style guide<style_guide_natspec>`. They are written with a
triple slash (``///``) or a double asterisk block (``/** ... */``) and
they should be used directly above function declarations or statements.

추가적으로 NatSpec comment 라고 불리는 다른 종류의 주석이 있는데, 이 주석은
:ref:`style guide<style_guide_natspec>`에 자세히 나와 있습니다. 이 주석은 
triple slash (``///``) 또는 double asterisk block (``/** ... */``)으로 작성되며
함수 선언이나 문장 바로 위에 사용해야 합니다.