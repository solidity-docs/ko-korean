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

컴파일러는 라이센스가 `SPDX에서 혀용하는 목록 <https://spdx.org/licenses/>`_의 일부인지는
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

``pragma`` 키워드는 특정 컴파일러 기능 또는 검사를 활성하하는 데에 사용됩니다.
pragma 지시문은 항상 소스파일에만 종속되어서, 만약 모든 프로젝트의
모든 파일에 pragma를 활성화하고 싶다면 이를 모든 파일에 추가해야합니다. 
.. index:: ! pragma, version

.. _version_pragma:

Version Pragma
버전 Pragma
--------------

Source files can (and should) be annotated with a version pragma to reject
compilation with future compiler versions that might introduce incompatible
changes. We try to keep these to an absolute minimum and
introduce them in a way that changes in semantics also require changes
in the syntax, but this is not always possible. Because of this, it is always
a good idea to read through the changelog at least for releases that contain
breaking changes. These releases always have versions of the form
``0.x.0`` or ``x.0.0``.

소스파일은 버전 Pragma를 주석으로 달아 호환되지 않는 변경사항을 도입할 지도 모르는
미래 컴파일러 버전과의 컴파일을 거부할 수 있습니다(해야만 합니다.).
우리는 이 변경사항을 절대적으로 최소화하고, 의미론의 변화 또는 구문의 변화를
요구하는 방식으로 도입하기 위해 노력하고 있습니다만, 이것이 항상
가능한 것은 아닙니다. 따라서 최소한의 변경 사항이 포함된 release의 경우
changelog를 통해 읽는 것이 항상 좋은 방법입니다. 이러한 release는 항상 
``0.x.0`` 또는 ``x.0.0`` 형식의 버전이 있습니다.

The version pragma is used as follows: ``pragma solidity ^0.5.2;``
버전 pragma는 다음과 같이 사용됩니다. ``pragma solidity ^0.5.2;``

A source file with the line above does not compile with a compiler earlier than version 0.5.2,
and it also does not work on a compiler starting from version 0.6.0 (this
second condition is added by using ``^``). Because
there will be no breaking changes until version ``0.6.0``, you can
be sure that your code compiles the way you intended. The exact version of the
compiler is not fixed, so that bugfix releases are still possible.

위 라인이 쓰여진 소스파일은 0.5.2 보다 이전 버전의 컴파일러로 컴파일되지 않고,
버전 0.6.0으로 시작하는 컴파일 위에서 동작하지 않습니다(두 번째 조건은 ``^``를 사용하여 추가됩니다.).
``0.6.0``버전까지는 변경사항이 없을 것이기 때문에, 코드가 의도한 대로 컴파일될 것임을 확신할 수 있습니다.
컴파일러의 정확한 버전은 고정되지 않아서, 버그 수정 release는 언제나 가능합니다.

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

  버전 pragma를 사용하는 것은 컴파일러의 버전을 *바꾸는* 것이 아닙니다.
  또한 컴파일러의 특징을 *활성화/비활성화* 하는 것도 아닙니다.
  단지 pragma에 의해 요구된 버전에 부합하는지 확인하기 위해 컴파일러에게 알려주는 것 뿐입니다.
  버전이 맞지 않다면, 컴파일러는 에러를 출력합니다.

ABI Coder Pragma
----------------

By using ``pragma abicoder v1`` or ``pragma abicoder v2`` you can
select between the two implementations of the ABI encoder and decoder.

The new ABI coder (v2) is able to encode and decode arbitrarily nested
arrays and structs. It might produce less optimal code and has not
received as much testing as the old encoder, but is considered
non-experimental as of Solidity 0.6.0. You still have to explicitly
activate it using ``pragma abicoder v2;``. Since it will be
activated by default starting from Solidity 0.8.0, there is the option to select
the old coder using ``pragma abicoder v1;``.

The set of types supported by the new encoder is a strict superset of
the ones supported by the old one. Contracts that use it can interact with ones
that do not without limitations. The reverse is possible only as long as the
non-``abicoder v2`` contract does not try to make calls that would require
decoding types only supported by the new encoder. The compiler can detect this
and will issue an error. Simply enabling ``abicoder v2`` for your contract is
enough to make the error go away.

.. note::
  This pragma applies to all the code defined in the file where it is activated,
  regardless of where that code ends up eventually. This means that a contract
  whose source file is selected to compile with ABI coder v1
  can still contain code that uses the new encoder
  by inheriting it from another contract. This is allowed if the new types are only
  used internally and not in external function signatures.

.. note::
  Up to Solidity 0.7.4, it was possible to select the ABI coder v2
  by using ``pragma experimental ABIEncoderV2``, but it was not possible
  to explicitly select coder v1 because it was the default.

.. index:: ! pragma, experimental

.. _experimental_pragma:

Experimental Pragma
-------------------

The second pragma is the experimental pragma. It can be used to enable
features of the compiler or language that are not yet enabled by default.
The following experimental pragmas are currently supported:


ABIEncoderV2
~~~~~~~~~~~~

Because the ABI coder v2 is not considered experimental anymore,
it can be selected via ``pragma abicoder v2`` (please see above)
since Solidity 0.7.4.

.. _smt_checker:

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

If you use ``pragma experimental SMTChecker;``, then you get additional
:ref:`safety warnings<formal_verification>` which are obtained by querying an
SMT solver.
The component does not yet support all features of the Solidity language and
likely outputs many warnings. In case it reports unsupported features, the
analysis may not be fully sound.

.. index:: source file, ! import, module, source unit

.. _import:

Importing other Source Files
============================

Syntax and Semantics
--------------------

Solidity supports import statements to help modularise your code that
are similar to those available in JavaScript
(from ES6 on). However, Solidity does not support the concept of
a `default export <https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Description>`_.

At a global level, you can use import statements of the following form:

.. code-block:: solidity

    import "filename";

The ``filename`` part is called an *import path*.
This statement imports all global symbols from "filename" (and symbols imported there) into the
current global scope (different than in ES6 but backwards-compatible for Solidity).
This form is not recommended for use, because it unpredictably pollutes the namespace.
If you add new top-level items inside "filename", they automatically
appear in all files that import like this from "filename". It is better to import specific
symbols explicitly.

The following example creates a new global symbol ``symbolName`` whose members are all
the global symbols from ``"filename"``:

.. code-block:: solidity

    import * as symbolName from "filename";

which results in all global symbols being available in the format ``symbolName.symbol``.

A variant of this syntax that is not part of ES6, but possibly useful is:

.. code-block:: solidity

  import "filename" as symbolName;

which is equivalent to ``import * as symbolName from "filename";``.

If there is a naming collision, you can rename symbols while importing. For example,
the code below creates new global symbols ``alias`` and ``symbol2`` which reference
``symbol1`` and ``symbol2`` from inside ``"filename"``, respectively.

.. code-block:: solidity

    import {symbol1 as alias, symbol2} from "filename";

.. index:: virtual filesystem, source unit name, import; path, filesystem path, import callback, Remix IDE

Import Paths
------------

In order to be able to support reproducible builds on all platforms, the Solidity compiler has to
abstract away the details of the filesystem where source files are stored.
For this reason import paths do not refer directly to files in the host filesystem.
Instead the compiler maintains an internal database (*virtual filesystem* or *VFS* for short) where
each source unit is assigned a unique *source unit name* which is an opaque and unstructured identifier.
The import path specified in an import statement is translated into a source unit name and used to
find the corresponding source unit in this database.

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

For a complete description of the virtual filesystem and the path resolution logic used by the
compiler see :ref:`Path Resolution <path-resolution>`.

.. index:: ! comment, natspec

Comments
========

Single-line comments (``//``) and multi-line comments (``/*...*/``) are possible.

.. code-block:: solidity

    // This is a single-line comment.

    /*
    This is a
    multi-line comment.
    */

.. note::
  A single-line comment is terminated by any unicode line terminator
  (LF, VF, FF, CR, NEL, LS or PS) in UTF-8 encoding. The terminator is still part of
  the source code after the comment, so if it is not an ASCII symbol
  (these are NEL, LS and PS), it will lead to a parser error.

Additionally, there is another type of comment called a NatSpec comment,
which is detailed in the :ref:`style guide<style_guide_natspec>`. They are written with a
triple slash (``///``) or a double asterisk block (``/** ... */``) and
they should be used directly above function declarations or statements.
