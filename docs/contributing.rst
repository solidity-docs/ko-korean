############
기여하기
############

도움은 항상 환영하며 Solidity에 기여하는 방법은 여러가지가 있습니다.

특히 다음 분야에서 많은 도움을 기다리고 있습니다:

* 이슈 리포트
* `Solidity의 깃허브 이슈
  <https://github.com/ethereum/solidity/issues>`_ 수정 및 대응, 특히 외부 컨트리뷰터의 입문용 이슈를 뜻하는 
  `"good first issue" <https://github.com/ethereum/solidity/labels/good%20first%20issue>`_ 라고 태그된 이슈들
* 문서 개선
* 다른 언어로 문서 번역
* `StackExchange
  <https://ethereum.stackexchange.com>`_ 와 `Solidity Gitter Chat
  <https://gitter.im/ethereum/solidity>`_ 에서 다른 사용자들의 질문에 답변
* `Solidity forum <https://forum.soliditylang.org/>`_ 언어 변경사항 혹은 새로운 피쳐를 제안하거나 피드백을 제공함으로써 언어 설계 과정에 참여

먼저 솔리디티의 컴포넌트와 빌드 프로세스에 익숙해지려면 :ref:`building-from-source` 
를 참고하십시오. 또한 솔리디티로 스마트 컨트랙트를 작성하는 방법을 숙지하는 것도 좋습니다.

이 프로젝트는 다음 `컨트리뷰터 행동 지침 <https://raw.githubusercontent.com/ethereum/solidity/develop/CODE_OF_CONDUCT.md>`_ 
와 함께 릴리즈되었습니다. 프로젝트에 참여하면 (이슈, 풀리퀘스트, Gitter 채널 등) 이에 동의하게 됩니다.

Team Calls
==========

논의가 필요한 이슈 또는 풀리퀘스트가 있거나, 혹은 팀 또는 컨트리뷰터들이
무엇을 하고 있는지 듣고 싶으면 다음 공개 팀 콜에 참여하실 수 있습니다:

- 월요일 3pm CET/CEST.
- 수요일 2pm CET/CEST.

둘 다 `Jitsi <https://meet.ethereum.org/solidity>`_ 에서 참여할 수 있습니다.

이슈 리포팅
====================

이슈 리포팅은
`GitHub issues tracker <https://github.com/ethereum/solidity/issues>`_ 를 사용하십시오.
이슈 리포팅 시 다음과 같은 세부사항들을 알려주십시오:

* 솔리디티 버전
* 소스코드 (가능한 경우)
* 운영체제
* 이슈 재현을 위한 단계
* 실제 작동과 의도한 작동 비교

이슈를 발생시킨 소스코드를 최소한으로 줄이는 것 항상 많은 도움이 되며
때로는 오해를 해결하기도 합니다.

풀리퀘스트 워크플로우
==========================

기여하기 위해서는 ``develop`` 브랜치를 포크하고 거기서 수정하십시오. 커밋 메시지는
*무엇* 을 변경했는 지 외에 *왜* 변경을 했는 지도 기술해야 합니다. (작은 변경이 아닌 경우)

포크 이후 ``develop`` 에서 변경사항을 pull해야하면 (가령 잠재적 병합 충돌을 해결), 
``git merge`` 사용은 피하고 대신 작업 브랜치를 ``git rebase`` 하십시오. 이는 변경사항
리뷰를 더 쉽게 합니다.

또한, 새로운 피쳐를 작성 중이라면, ``test/`` 하단에 필요한 테스트 케이스를 반드시 추가하십시오 (아래 참고).

다만, 더 큰 변경을 하는 경우, `Solidity Development Gitter channel <https://gitter.im/ethereum/solidity-dev>`_
과 먼저 논의하십히오 (상기의 채널과 별개로, 이는 언어의 사용이 아닌 컴파일러와 언어 개발에 중점을 두고 있습니다).

새로운 피쳐와 버그픽스 내역는 ``Changelog.md`` 파일에 추가되어야 합니다.
해당하는 경우 이전 엔트리의 스타일을 따르십시오.

마지막으로, 우리 프로젝트의 `coding style
<https://github.com/ethereum/solidity/blob/develop/CODING_STYLE.md>`_ 을 존중해주십시오.
또한, CI 테스팅을 하고 있지만, 풀리퀘스트를 제출하기 전에 코드를 테스트하고 로컬에서 빌드되는 지 확인하십시오.

도움을 주셔서 감사합니다!

Running the Compiler Tests
==========================

Prerequisites
-------------

모든 컴파일러 테스트를 실행하기 위해서는 몇몇 의존성을 선택적으로 설치할 수 있습니다.
(`evmone <https://github.com/ethereum/evmone/releases>`_, `libz3 <https://github.com/Z3Prover/z3>`_,
`libhera <https://github.com/ewasm/hera>`_).

MacOS의 경우 일부 테스트 스크립트는 GNU coreutils가 설치되어야 합니다.
이는 Homebrew를 사용하는게 가장 쉬울 수 있습니다: ``brew install coreutils``.

Running the Tests
-----------------

Solidity는 여러 종류의 테스트가 있으며, 대부분은 `Boost C++ Test Framework
<https://www.boost.org/doc/libs/release/libs/test/doc/html/index.html>`_ 어플리케이션 ``soltest`` 에 번들로 제공됩니다.
대부분의 변경의 경우 ``build/test/soltest`` 또는 이의 wrapper인 ``scripts/soltest.sh`` 를 실행하면 충분합니다.

``./scripts/tests.sh`` 스크립트는 `Boost C++ Test Framework <https://www.boost.org/doc/libs/release/libs/test/doc/html/index.html>`_
어플리케이션 ``soltest`` (또는 wrapper ``scripts/soltest.sh``)에 번들된 테스트, 그리고 커멘드라인 테스트와 컴파일 테스트를 포함해서
대부분의 Solidity 테스트를 자동으로 실행합니다.

테스트 시스템은 semantic 테스트를 실행하기 위해 `evmone <https://github.com/ethereum/evmone/releases>`_
의 위치를 자동으로 파악하려 합니다.

``evmone`` 라이브러리는 반드시 현재 작업 디렉토리, 또는 이의 부모, 또는 이의 부모의 부모에서 상대경로로
``deps`` 또는 ``deps/lib`` 디렉토리에 위치해야합니다. 대안으로 ``evmone`` 공유 객체에 대한 명시적 위치를
``ETH_EVMONE`` 환경변수에 지정할 수 있습니다.

``evmone`` 주로 semantic 과 가스 테스트 실행에 필요합니다.
설치하지 않은 경우, ``scripts/soltest.sh`` 에 ``--no-semantic-tests`` 플래그를 추가해 해당하는 테스트를 건너뛸 수 있습니다.

Ewasm 테스트는 기본적으로 비활성화되어 있으며, ``./scripts/soltest.sh --ewasm`` 를 사용해 명시적으로 허용할 수 있고
``soltest`` 가 이를 찾기 위해 `hera <https://github.com/ewasm/hera>`_ 를 요구합니다.
``hera`` 라이브러리를 위치시키는 방법은 명시적 위치 지정을 위한 변수 이름이 ``ETH_HERA`` 것을 제외하고 ``evmone`` 과 동일합니다.

``evmone`` 과 ``hera`` 라이브러리는 둘다 리눅스에서 ``.so``, 윈도우에서는 ``.dll``, MacOS에서는 ``.dylib`` 확장자로 끝나야 합니다.

SMT 테스트를 실행하려면 ``libz3`` 라이브러리가 설치되어 있어야 하며 컴파일러의 configure 단계에서 ``cmake`` 로
찾을 수 있어야 합니다.

``libz3`` 라이브러리가 설치되어있지 않다면 ``./scripts/tests.sh`` 를 실행하기 전에 ``SMT_FLAGS=--no-smt`` 를 익스포트해
SMT 테스트를 비활성화하거나 ``./scripts/soltest.sh --no-smt`` 를 실행하십시오.
SMT 테스트는 ``libsolidity/smtCheckerTests`` 과 ``libsolidity/smtCheckerTestsJSON`` 입니다.

.. note ::

    Soltest가 실행하는 모든 단위테스트 목록을 보기 위해서는 ``./build/test/soltest --list_content=HRF`` 를 실행하십시오.

더 빠른 결과 확인을 위해서는 부분 또는 특정 테스트를 실행할 수 있습니다.

테스트 하위 집합을 실행하려면 필터를 사용할 수 있습니다:
``./scripts/soltest.sh -t TestSuite/TestName``,
여기서 ``TestName`` 은 와일드카드 ``*`` 일 수 있습니다.

또는 가령 yul disambiguator의 모든 테스트를 실행하는 경우:
``./scripts/soltest.sh -t "yulOptimizerTests/disambiguator/*" --no-smt``.

``./build/test/soltest --help`` 는 가능한 모든 옵션에 대한 광범위 도움말을 보여줍니다.

특히 다음을 확인해보십시오:

- 테스트 컴파일레이션을 표시하기 위해서는 `show_progress (-p) <https://www.boost.org/doc/libs/release/libs/test/doc/html/boost_test/utf_reference/rt_param_reference/show_progress.html>`_ 
- 특정 테스트 케이스 실행을 위해서는 `run_test (-t) <https://www.boost.org/doc/libs/release/libs/test/doc/html/boost_test/utf_reference/rt_param_reference/run_test.html>`_
- 더 자세한 결과 리포트를 위해서는 `report-level (-r) <https://www.boost.org/doc/libs/release/libs/test/doc/html/boost_test/utf_reference/rt_param_reference/report_level.html>`_

.. note ::

    윈도우 환경에서 위 기본 세트를 libz3 없이 실행하고 싶은 경우 다음을 참고하십시오.
    Git Bash를 사용하는 경우, ``./build/test/Release/soltest.exe -- --no-smt`` 를 사용하십시오.
    일반 명령 프롬프트에서 실행하는 경우 ``.\build\test\Release\soltest.exe -- --no-smt`` 를 사용하십시오.

GDB를 사용해서 디버깅하려는 경우 "일반적인 경우"와 다르게 빌드해야 합니다.
예를 들어, 다음 명령어를 ``build`` 폴더에서 실행시킵니다:

.. code-block:: bash

   cmake -DCMAKE_BUILD_TYPE=Debug ..
   make

이는 심볼을 생성해 ``--debug`` 플래그를 사용해서 테스트 디버깅을 할 때
함수와 변수에 접근해 브레이크 또는 출력할 수 있게 합니다.

CI는 Emscripten 타깃 컴파일을 요구하는 추가적인 테스트를 실행합니다. (``solc-js`` 와 프레임워크 서드파티 테스트 포함)

Writing and Running Syntax Tests
--------------------------------

신텍스 테스트는 컴파일러가 유효하지 않는 코드에 대해 올바른 오류 메시지를 생성하고 유효한 코드를 올바르게
수락하는 지 확인합니다. 이는 ``tests/libsolidity/syntaxTests`` 폴더 안 개별 파일에 저장됩니다.
이 파일들은 반드시 각 테스트에서 예상하는 결과(들)을 설명하는 주석을 포함해야합니다.
테스트 스위트는 테스트들을 컴파일하고 주어진 예상 결과값들과 비교해 확인합니다.

예를 들어: ``./test/libsolidity/syntaxTests/double_stateVariable_declaration.sol``

.. code-block:: solidity

    contract test {
        uint256 variable;
        uint128 variable;
    }
    // ----
    // DeclarationError: (36-52): Identifier already declared.

신텍스 테스트는 최소한 테스트 중인 컨트랙트 자신을 반드시 포함해야 하며 이어서 ``// ----`` 로 분리합니다. 분리자 다음에 작성하는 주석은
예상되는 컴파일러 오류나 경고를 서술하는 데 사용합니다. 위 숫자 구간은 오류가 발생한 소스 위치를 의미합니다.
오류나 경고 없이 컨트랙트를 컴파일하기를 원한다면 분리자와 주석을 제거합니다.

위 예시에서, 상태 변수 ``variable`` 이 두 번 선언되어 있지만 이는 금지되어 있습니다.
이는 식별자가 이미 선언되었다는 ``DeclarationError`` 를 반환합니다.

``isoltest`` 툴은 이와 같은 테스트에 사용되며 ``./build/test/tools/`` 하단에서 발견할 수 있습니다. 이는 원하는 에디터를 사용해
실패하는 컨트랙트를 편집할 수 있도록 하는 대화식 툴입니다. 두번째 ``variable`` 선언을 지워 테스트가 실패하도록 해보겠습니다:

.. code-block:: solidity

    contract test {
        uint256 variable;
    }
    // ----
    // DeclarationError: (36-52): Identifier already declared.

``./build/test/tools/isoltest`` 를 다시 실행하면 테스트가 실패합니다:

.. code-block:: text

    syntaxTests/double_stateVariable_declaration.sol: FAIL
        Contract:
            contract test {
                uint256 variable;
            }

        Expected result:
            DeclarationError: (36-52): Identifier already declared.
        Obtained result:
            Success


``isoltest`` 는 예상 결과값과 실제 얻은 결과값을 출력하고, 현재 컨트랙트 파일을 수정, 업데이트, 또는 건너뛰거나,
또는 어플리케이션 종료하는 방법를 제공합니다.

실패하는 테스트에 대해 몇가지 옵션을 제공합니다:

- ``edit``: ``isoltest`` 가 수정할 수 있도록 에디터에 컨트랙트를 열려고 시도합니다. 이는 커멘드 라인 (``isoltest --editor /path/to/editor``),
  ``EDITOR`` 환경변수, 또는 ``/usr/bin/editor`` 를 순서대로 참고해서 에디터를 선택합니다.
- ``update``: 테스트 중인 컨트랙트의 예상 결과값을 업데이트합니다. 이는 충족하지 않은 결과값을 지우고 누락된 결과값을 추가하는 방식으로 주석을
  업데이트합니다. 그 후 테스트를 다시 실행합니다.
- ``skip``: 해당 특정 테스트를 건너뜁니다.
- ``quit``: ``isoltest`` 를 종료합니다.

테스트 프로세스를 종료하는 ``quit`` 를 제외하고 위 모든 옵션들은 현재 컨트랙트에 적용됩니다.

위 테스트를 자동으로 업데이트하면 다음과 같이 바뀝니다.

.. code-block:: solidity

    contract test {
        uint256 variable;
    }
    // ----

그리고 테스트를 재실행하면 이제는 다시 통과합니다:

.. code-block:: text

    Re-running test case...
    syntaxTests/double_stateVariable_declaration.sol: OK


.. note::

    컨트랙트 파일의 이름은 ``double_variable_declaration.sol`` 와 같이 무엇을 테스트하는 지 설명하도록 작성하십시오.
    상속이나 cross-contract call을 테스트하는 경우가 아니라면 하나의 파일에 둘 이상의 컨트랙트를 포함하지 마십시오.
    각 파일은 새로운 기능의 한 측면을 테스트해야 합니다.


Running the Fuzzer via AFL
==========================

Fuzzing is a technique that runs programs on more or less random inputs to find exceptional execution
states (segmentation faults, exceptions, etc). Modern fuzzers are clever and run a directed search
inside the input. We have a specialized binary called ``solfuzzer`` which takes source code as input
and fails whenever it encounters an internal compiler error, segmentation fault or similar, but
does not fail if e.g., the code contains an error. This way, fuzzing tools can find internal problems in the compiler.

We mainly use `AFL <https://lcamtuf.coredump.cx/afl/>`_ for fuzzing. You need to download and
install the AFL packages from your repositories (afl, afl-clang) or build them manually.
Next, build Solidity (or just the ``solfuzzer`` binary) with AFL as your compiler:

.. code-block:: bash

    cd build
    # if needed
    make clean
    cmake .. -DCMAKE_C_COMPILER=path/to/afl-gcc -DCMAKE_CXX_COMPILER=path/to/afl-g++
    make solfuzzer

At this stage you should be able to see a message similar to the following:

.. code-block:: text

    Scanning dependencies of target solfuzzer
    [ 98%] Building CXX object test/tools/CMakeFiles/solfuzzer.dir/fuzzer.cpp.o
    afl-cc 2.52b by <lcamtuf@google.com>
    afl-as 2.52b by <lcamtuf@google.com>
    [+] Instrumented 1949 locations (64-bit, non-hardened mode, ratio 100%).
    [100%] Linking CXX executable solfuzzer

If the instrumentation messages did not appear, try switching the cmake flags pointing to AFL's clang binaries:

.. code-block:: bash

    # if previously failed
    make clean
    cmake .. -DCMAKE_C_COMPILER=path/to/afl-clang -DCMAKE_CXX_COMPILER=path/to/afl-clang++
    make solfuzzer

Otherwise, upon execution the fuzzer halts with an error saying binary is not instrumented:

.. code-block:: text

    afl-fuzz 2.52b by <lcamtuf@google.com>
    ... (truncated messages)
    [*] Validating target binary...

    [-] Looks like the target binary is not instrumented! The fuzzer depends on
        compile-time instrumentation to isolate interesting test cases while
        mutating the input data. For more information, and for tips on how to
        instrument binaries, please see /usr/share/doc/afl-doc/docs/README.

        When source code is not available, you may be able to leverage QEMU
        mode support. Consult the README for tips on how to enable this.
        (It is also possible to use afl-fuzz as a traditional, "dumb" fuzzer.
        For that, you can use the -n option - but expect much worse results.)

    [-] PROGRAM ABORT : No instrumentation detected
             Location : check_binary(), afl-fuzz.c:6920


Next, you need some example source files. This makes it much easier for the fuzzer
to find errors. You can either copy some files from the syntax tests or extract test files
from the documentation or the other tests:

.. code-block:: bash

    mkdir /tmp/test_cases
    cd /tmp/test_cases
    # extract from tests:
    path/to/solidity/scripts/isolate_tests.py path/to/solidity/test/libsolidity/SolidityEndToEndTest.cpp
    # extract from documentation:
    path/to/solidity/scripts/isolate_tests.py path/to/solidity/docs

The AFL documentation states that the corpus (the initial input files) should not be
too large. The files themselves should not be larger than 1 kB and there should be
at most one input file per functionality, so better start with a small number of.
There is also a tool called ``afl-cmin`` that can trim input files
that result in similar behaviour of the binary.

Now run the fuzzer (the ``-m`` extends the size of memory to 60 MB):

.. code-block:: bash

    afl-fuzz -m 60 -i /tmp/test_cases -o /tmp/fuzzer_reports -- /path/to/solfuzzer

The fuzzer creates source files that lead to failures in ``/tmp/fuzzer_reports``.
Often it finds many similar source files that produce the same error. You can
use the tool ``scripts/uniqueErrors.sh`` to filter out the unique errors.

Whiskers
========

*Whiskers* is a string templating system similar to `Mustache <https://mustache.github.io>`_. It is used by the
compiler in various places to aid readability, and thus maintainability and verifiability, of the code.

The syntax comes with a substantial difference to Mustache. The template markers ``{{`` and ``}}`` are
replaced by ``<`` and ``>`` in order to aid parsing and avoid conflicts with :ref:`yul`
(The symbols ``<`` and ``>`` are invalid in inline assembly, while ``{`` and ``}`` are used to delimit blocks).
Another limitation is that lists are only resolved one depth and they do not recurse. This may change in the future.

A rough specification is the following:

Any occurrence of ``<name>`` is replaced by the string-value of the supplied variable ``name`` without any
escaping and without iterated replacements. An area can be delimited by ``<#name>...</name>``. It is replaced
by as many concatenations of its contents as there were sets of variables supplied to the template system,
each time replacing any ``<inner>`` items by their respective value. Top-level variables can also be used
inside such areas.

There are also conditionals of the form ``<?name>...<!name>...</name>``, where template replacements
continue recursively either in the first or the second segment depending on the value of the boolean
parameter ``name``. If ``<?+name>...<!+name>...</+name>`` is used, then the check is whether
the string parameter ``name`` is non-empty.

.. _documentation-style:

Documentation Style Guide
=========================

In the following section you find style recommendations specifically focusing on documentation
contributions to Solidity.

English Language
----------------

Use English, with British English spelling preferred, unless using project or brand names. Try to reduce the usage of
local slang and references, making your language as clear to all readers as possible. Below are some references to help:

* `Simplified technical English <https://en.wikipedia.org/wiki/Simplified_Technical_English>`_
* `International English <https://en.wikipedia.org/wiki/International_English>`_
* `British English spelling <https://en.oxforddictionaries.com/spelling/british-and-spelling>`_


.. note::

    While the official Solidity documentation is written in English, there are community contributed :ref:`translations`
    in other languages available. Please refer to the `translation guide <https://github.com/solidity-docs/translation-guide>`_
    for information on how to contribute to the community translations.

Title Case for Headings
-----------------------

Use `title case <https://titlecase.com>`_ for headings. This means capitalise all principal words in
titles, but not articles, conjunctions, and prepositions unless they start the
title.

For example, the following are all correct:

* Title Case for Headings.
* For Headings Use Title Case.
* Local and State Variable Names.
* Order of Layout.

Expand Contractions
-------------------

Use expanded contractions for words, for example:

* "Do not" instead of "Don't".
* "Can not" instead of "Can't".

Active and Passive Voice
------------------------

Active voice is typically recommended for tutorial style documentation as it
helps the reader understand who or what is performing a task. However, as the
Solidity documentation is a mixture of tutorials and reference content, passive
voice is sometimes more applicable.

As a summary:

* Use passive voice for technical reference, for example language definition and internals of the Ethereum VM.
* Use active voice when describing recommendations on how to apply an aspect of Solidity.

For example, the below is in passive voice as it specifies an aspect of Solidity:

  Functions can be declared ``pure`` in which case they promise not to read
  from or modify the state.

For example, the below is in active voice as it discusses an application of Solidity:

  When invoking the compiler, you can specify how to discover the first element
  of a path, and also path prefix remappings.

Common Terms
------------

* "Function parameters" and "return variables", not input and output parameters.

Code Examples
-------------

A CI process tests all code block formatted code examples that begin with ``pragma solidity``, ``contract``, ``library``
or ``interface`` using the ``./test/cmdlineTests.sh`` script when you create a PR. If you are adding new code examples,
ensure they work and pass tests before creating the PR.

Ensure that all code examples begin with a ``pragma`` version that spans the largest where the contract code is valid.
For example ``pragma solidity >=0.4.0 <0.9.0;``.

Running Documentation Tests
---------------------------

Make sure your contributions pass our documentation tests by running ``./scripts/docs.sh`` that installs dependencies
needed for documentation and checks for any problems such as broken links or syntax issues.

Solidity Language Design
========================

To actively get involved in the language design process and share your ideas concerning the future of Solidity,
please join the `Solidity forum <https://forum.soliditylang.org/>`_.

The Solidity forum serves as the place to propose and discuss new language features and their implementation in
the early stages of ideation or modifications of existing features.

As soon as proposals get more tangible, their
implementation will also be discussed in the `Solidity GitHub repository <https://github.com/ethereum/solidity>`_
in the form of issues.

In addition to the forum and issue discussions, we regularly host language design discussion calls in which selected
topics, issues or feature implementations are debated in detail. The invitation to those calls is shared via the forum.

We are also sharing feedback surveys and other content that is relevant to language design in the forum.

If you want to know where the team is standing in terms or implementing new features, you can follow the implementation status in the `Solidity Github project <https://github.com/ethereum/solidity/projects/43>`_.
Issues in the design backlog need further specification and will either be discussed in a language design call or in a regular team call. You can
see the upcoming changes for the next breaking release by changing from the default branch (`develop`) to the `breaking branch <https://github.com/ethereum/solidity/tree/breaking>`_.

For ad-hoc cases and questions you can reach out to us via the `Solidity-dev Gitter channel <https://gitter.im/ethereum/solidity-dev>`_, a
dedicated chatroom for conversations around the Solidity compiler and language development.

We are happy to hear your thoughts on how we can improve the language design process to be even more collaborative and transparent.
