.. index:: ! installing

.. _installing-solidity:

################################
Solidity 컴파일러 설치하기
################################

버저닝
==========

Solidity 버전들은 `semantic versioning <https://semver.org>`_ 방식을 따르며 **nightly 빌드** 또한 가능합니다. 
nightly 빌드는 항상 동작한다고 보기에는 힘들며,  
The nightly builds
are not guaranteed to be working and despite best efforts they might contain undocumented
and/or broken changes. We recommend using the latest release. Package installers below
will use the latest release.

Remix
=====

*규모가 작은 컨트랙트를 작성하거나 Solidity를 보다 빠르게 배우기 위해 Remix를 사용할 것을 추천드립니다.*

`Remix online에 접속하면 <https://remix.ethereum.org/>`_, 어떤 것도 설치하실 필요가 없어집니다.
인터넷 연결 없이 사용하고 싶으시다면, https://github.com/ethereum/remix-live/tree/gh-pages 페이지에 접속하신 후 ``.zip`` 파일을 다운로드 하십시오.
Remix는 여러 버전의 Solidity를 설치하지 않고도 nightly 빌드를 테스트해볼 수 있는 편리한 옵션이기도 합니다. 

이 페이지에선 여러분의 컴퓨터에 Solidity 컴파일러 소프트웨어 커맨드라인을 설치하기 위한 자세한 옵션들을 다뤄볼 예정입니다. 
규모가 큰 컨트랙트나 더 많은 컴파일 옵션이 필요하실 경우 커맨드라인 컴파일러를 사용해 보십시오. 

.. _solcjs:

npm / Node.js
=============

``npm`` 을 통해 Solidity 컴파일러인 ``solcjs`` 를 보다 편리하게 설치해보세요.
`solcjs` 프로그램은 이 페이지 하단 부분에 소개된 컴파일러로 접근하는 것보다는 기능들이 적습니다. 
:ref:`commandline-compiler` 문서는 여러분들이 모든 기능을 포함하고 있는 컴파일러인 ``solc``를 사용하고 있다고 가정합니다. 
이 `레포지토리 <https://github.com/ethereum/solc-js>`_ 안에 ``solcjs`` 에 대한 자세한 설명이 있습니다. 

참고: solc-js 프로젝트는 Emscripten, 즉 같은 컴파일러 소스 코드를 사용하는 C++ `solc` 에서 유래됐습니다. 
`solc-js` 는 Remix처럼 JavaScript 프로젝트에 사용될 수 있습니다. 
자세한 사항은 solc-js 레포지토리를 참고해주십시오.

.. code-block:: bash

    npm install -g solc

.. 참고::

    commandline executable은 ``solcjs`` 라 불립니다.

    ``solcjs`` 커맨드라인 옵션은 ``solc`` 및 툴들(예: ``geth``)과 호환되지 않습니다. 
    따라서 ``solc`` 에서의 행동은 ``solcjs`` 에선 작동되지 않습니다. 

Docker
======

Solidity 빌드의 Docker 이미지들은 ``ethereum`` 단체의 ``solc`` 이미지를 통해 가능합니다.
가장 최신 버전은 ``stable`` 태그를, 잠재적으로 불안정한 변동은 develop 브랜치에 있는 ``nightly`` 를 사용하십시오.

Docker 이미지는 compiler executable를 실행하기 때문에 모든 컴파일러 인수를 전달할 수 있습니다. 
예를 들어, (만약 여러분이 가지고 있지 않다면) 아래 명령어가 안정된 버전의 ``solc`` 이미지를 ``--help`` 인수를 전달한 후 pull하여 새로운 컨테이너에서 작동시키게 합니다.

.. code-block:: bash

    docker run ethereum/solc:stable --help

0.5.4 버전처럼 여러분들이 원하는 빌드 버전을 태그를 명시할 수도 있습니다.

.. code-block:: bash

    docker run ethereum/solc:0.5.4 --help

Docker 이미지를 호스트 머신에서 Solidity 파일들을 컴파일하려면 입력과 출력을 위한 로컬 폴더를 불러온 뒤 컴파일 하고자 하는 컨트랙트를 지정합니다. 예를 들자면,

.. code-block:: bash

    docker run -v /local/path:/sources ethereum/solc:stable -o /sources/output --abi --bin /sources/Contract.sol

툴링과 함께 컴파일러를 사용할 때 추천드리는 표준 JSON 인터페이스를 사용하실 수도 있습니다. 
이 인터페이스를 사용할 땐 JSON 입력이 self-contained되어 있을 경우 어떤 경로도 불러오실 필요가 없습니다.
:ref:`import callback에 의해 로드되어야 하는 <initial-vfs-content-standard-json-with-import-callback>` 어떠한 외부 파일을 참조할 필요가 없습니다.

.. code-block:: bash

    docker run ethereum/solc:stable --standard-json < input.json > output.json

리눅스 패키지
==============

Solidity의 Binary 패키지들은 `solidity/releases <https://github.com/ethereum/solidity/releases>`_ 에서 확인 가능합니다.

Ubuntu를 위한 PPA 또한 있지만 다음 명령어를 통해서 최신의 안정화 버전을 받으실 수 있습니다.

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc

Nightly 버전의 경우 다음 명령어를 통해 설치됩니다.

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo add-apt-repository ppa:ethereum/ethereum-dev
    sudo apt-get update
    sudo apt-get install solc

또한, 몇몇 리눅스 버전은 독자적인 패키지를 제공합니다. 이러한 패키지들은 저희가 직접 유지 보수를 하고 있진 않습니다만,
패키지를 유지 보수하는 사람들에 의해 계속해서 업데이트 되고 있습니다.

예를 들어, Arch 리눅스는 최신 개발 버전의 패키지를 가지고 있습니다.

.. code-block:: bash

    pacman -S solidity

`snap package <https://snapcraft.io/solc>`_ 라는 것도 있지만 **현재는 유지 보수가 되고 있지 않습니다**.
모든 `supported Linux distros <https://snapcraft.io/docs/core/install>`_ 내에서 설치가 가능합니다. 
solc의 가장 최신 안정화 버전을 설치하시려면, 

.. code-block:: bash

    sudo snap install solc

만일 여러분들께서 Solidity의 최신 개발 버전을 테스팅하는데 도움을 주시고 싶으시다면
다음을 시도해보십시오.

.. code-block:: bash

    sudo snap install solc --edge

.. 참고::

    ``solc`` 스냅은 엄격히 통제됩니다. 스냅 패키지에게 가장 보안이 뛰어난 모드로 제공되지만 ``/home`` 혹은 ``/media`` 와 같은 경로 안의 
    파일들만 접근하는 등의 제한이 걸리게 됩니다.
    자세한 사항은 `Demystifying Snap Confinement <https://snapcraft.io/blog/demystifying-snap-confinement>`_ 부분을 확인해주십시오.

macOS 패키지
==============

Solidity 컴파일러는 build-from-source 버전으로 Homebrew를 통해 제공됩니다.
Pre-built bottle는 현재 제공되고 있지 않습니다.

.. code-block:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity

Solidity 0.4.x / 0.5.x의 가장 최신 버전을 다운로드하기 위하여 ``brew install solidity@4`` 및 ``brew install solidity@5`` 를 사용하실 수 있습니다,

만일 특정 버전의 Solidity를 원하실 경우 Github에서 직접 Homebrew formula를 설치하실 수 있습니다.

`solidity.rb commits on Github <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_ 를 참조해주십시오.

여러분들께서 원하는 버전의 해시 커밋을 복사하신 후 컴퓨터에서 확인해보시길 바랍니다.

.. code-block:: bash

    git clone https://github.com/ethereum/homebrew-ethereum.git
    cd homebrew-ethereum
    git checkout <your-hash-goes-here>

``brew`` 를 이용하여 설치합니다.

.. code-block:: bash

    brew unlink solidity
    # eg. Install 0.4.8
    brew install solidity.rb

Static Binaries
===============

저희는 `solc-bin`_ 지원되는 모든 플랫폼을 위한 지난 혹은 현 컴파일러 버전의 스태틱 빌드를 포함하는 레포지토리를 운영하고 있습니다. 
여러분은 여기서 nightly 빌드 또한 찾아보실 수 있습니다.

이 레포지토리는 사용자들이 사용할 수 있는 바이너리들을 찾는 쉽고 빠른 방법일 뿐만이 아니라 다른 3자 툴과도 호환이 가능합니다. 

- 해당 콘텐츠는 https://binaries.soliditylang.org에 미러링되어 있으며 HTTPS, 인증, rate limiting 혹은 git을 사용하지 않고도 쉽게 다운로드 가능합니다.
- 콘텐트는 올바른 `Content-Type` 헤더를 통해 제공되며 CORS 설정에 비교적 업격하지 않아 브라우저에서 작동되는 툴에 의해 바로 로드될 수 있습니다.
- 바이너리들은 (필수 DLL과 함께 번들링된 오래된 Windows 빌드의 예외와 함께) 설치나 언팩킹이 필요 없습니다.
- 저희는 최고의 호환성을 유지하기 위해 노력하고 있습니다. 파일들은 한 번 추가되면 예전 위치에서 symlink나 redirect를 제공해주지 않으면 제거되거나 이동되지 않습니다. 
  파일들은 또한 절대 변경되지 않으며 반드시 원본 검사합과 항상 합치해야 합니다. 발생될 수 있는 유일한 예외는 깨졌거나 사용 불가능한 파일들이 가져올 수 있는 잠정적인 해입니다. 
- 파일들은 HTTP와 HTTPS를 통해 서브가 됩니다. 여러분들께서 파일 리스트를 (git, HTTPS, IPFS 혹은 로컬에서 캐싱함으로서) 안전한 방법으로 보관하고 
  파일 다운로드 후 바이너리들의 해시를 인증하실 수만 있다면, HTTPS를 사용하실 필요가 없습니다.

동일한 바이너리들은 대부분 `Solidity release page on Github`_ 상에서 가능합니다. 차이점은 저희가 Github 배포 페이지에서는 오래된 버전이 릴리즈에 대해서 업데이트를 하지 않는다는 점입니다.
이는 네이밍 컨벤션이 바뀔 경우 재명명하지 않고 릴리즈 당시 호환되지 않는 플랫폼들에 대한 빌드를 추가하지 않는다는 뜻입니다. 
이는 오직 ``solc-bin`` 에서만 이루어집니다.

``solc-bin`` 레포지토리는 몇 가지 상위 디렉토리를 가지고 있으며 각각의 디렉토리는 단일 플랫폼을 대표하고 있습니다. 
각각의 디렉토리들은 사용 가능한 바이너리들의 리스트인 ``list.json`` 파일을 가지고 있습니다. 
예를 들어 ``emscripten-wasm32/list.json`` 파일의 경우 버전 0.7.4에서 다음과 같은 정보를 확인하실 수 있습니다.

.. code-block:: json

    {
      "path": "solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js",
      "version": "0.7.4",
      "build": "commit.3f05b770",
      "longVersion": "0.7.4+commit.3f05b770",
      "keccak256": "0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3",
      "sha256": "0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2",
      "urls": [
        "bzzr://16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1",
        "dweb:/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS"
      ]
    }

이는 다음을 의미합니다.

- `solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js <https://github.com/ethereum/solc-bin/blob/gh-pages/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js>`_ 에서 여러분은 동일한 디렉토리에 있는 바이너리를 찾아보실 수 있습니다.
  파일은 symlink일 수 있기 때문에 git을 통해 다운로드하지 않을 경우 스스로 해결하셔야 하며 그렇지 않을 경우 파일은 symlink와 호환되지 않습니다.
- 바이너리는 또한 https://binaries.soliditylang.org/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js에 미러링되어 있습니다.
  이 경우는 파일의 복사본을 제공하거나 HTTP redirect를 반환하여 git이 필요하지 않고 symlink가 투명하게 해결될 경우를 의미합니다.
- 파일은 IPFS의 `QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS`_ 상에서 가능합니다. 
- 파일은 추후 Swarm의 `16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1`_ 에서도 가능해질 수 있습니다.
- 바이너리 무결성을 keccak256 해시와 ``0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3`` 와의 대조를 통해 인증할 수 있습니다.
  해시는 커맨드 라인에서 `sha3sum`_ 혹은 자바스크립트의 `keccak256() function from ethereumjs-util`_ 에 의해 제공되는 ``keccak256sum`` 유틸리티를 통해 연산될 수 있습니다.
- 바이너리 무결성을 sha256 해시와 ``0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2`` 를 통해서도 인증할 수 있습니다. 

.. 주의::

   하위 호환성으로 인해 이 레포지토리에는 몇몇 오래된 요소들을 포함하고 있어 새로운 툴들을 작성할 시 가급적 사용을 피해주시기 바랍니다.

   - 최고 성능을 원하신다면 ``bin/`` 대신 ``emscripten-wasm32/``(fallback ``emscripten-asmjs/``)을 사용하시기 바랍니다.
     0.6.1 버전 전까지 오로지 asm.js 바이너리들만 제공이 됩니다.
     0.6.2 버전 이후부터 더욱 개선된 성능과 함께 `WebAssembly builds`_ 로 전환하였습니다. 
     저희는 wasm을 위해 오래된 버전을 다시 재구성하였지만 원본 asm.js 파일들은 여전히 ``bin/`` 에 있습니다.
     새로운 파일들은 이름 충돌을 피하기 위해 별도의 디렉토리에 자리잡고 있습니다.
   - wasm 혹은 asm.js 바이너리를 받고 있는지 확실히 하기 위해선 
     ``bin/`` 와 ``wasm/`` 디렉토리 대신 ``emscripten-asmjs/`` 와 ``emscripten-wasm32/`` 디렉토리를 사용하시기 바랍니다.
   - Use  instead of ``list.js`` 와 ``list.txt`` 대신 ``list.json`` 을 사용하시기 바랍니다.
     JSON 리스트 형태는 오래된 정보와 함께 더 많은 것을 포함하고 있습니다.
   - https://solc-bin.ethereum.org 대신 https://binaries.soliditylang.org 를 사용하시기 바랍니다. 
     조금 더 간단하게 만들기 위해 새로운 ``soliditylang.org`` 도메인에 있는 컴파일러와 관련된 모든 것들을 옮겼으며, 이는 ``solc-bin`` 에도 적용이 됩니다.
     새로운 도메인을 사용하시는 것을 추천드리지만, 기존의 것 또한 여전히 지원이 되며 똑같은 위치에서 동작됨을 보장합니다.

.. 주의::

    바이너리들은 https://ethereum.github.io/solc-bin/ 에서도 확인이 가능하지만 0.7.2 버전 릴리즈 이후 더 이상 업데이트 되지 않습니다.
    이에 따라 어떠한 새로운 릴리즈나 nightly 빌드를 받아보실 수 없으며 non-emscripten 빌드를 포함한 새로운 디렉토리 구조를 제공받으실 수 없습니다.

    만일 이를 사용하고 계시다면 drop-in replacement인 https://binaries.soliditylang.org 로 전환하시기 바랍니다.
    이는 보다 투명한 방법으로 기존의 호스팅을 변화시켜주며 충돌을 최소화합니다. 
    저희가 더 이상 관리하지 않는 ``ethereum.github.io`` 도메인과는 다르게, ``binaries.soliditylang.org`` 는 장기적으로 동일한 URL 구조를 유지할 수 있도록 해줍니다.

.. _IPFS: https://ipfs.io
.. _Swarm: https://swarm-gateways.net/bzz:/swarm.eth
.. _solc-bin: https://github.com/ethereum/solc-bin/
.. _Solidity release page on github: https://github.com/ethereum/solidity/releases
.. _sha3sum: https://github.com/maandree/sha3sum
.. _keccak256() function from ethereumjs-util: https://github.com/ethereumjs/ethereumjs-util/blob/master/docs/modules/_hash_.md#const-keccak256
.. _WebAssembly builds: https://emscripten.org/docs/compiling/WebAssembly.html
.. _QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS: https://gateway.ipfs.io/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS
.. _16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1: https://swarm-gateways.net/bzz:/16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1/

.. _building-from-source:

Building from Source
====================

Prerequisites - All Operating Systems
-------------------------------------

The following are dependencies for all builds of Solidity:

+-----------------------------------+-------------------------------------------------------+
| Software                          | Notes                                                 |
+===================================+=======================================================+
| `CMake`_ (version 3.13+)          | Cross-platform build file generator.                  |
+-----------------------------------+-------------------------------------------------------+
| `Boost`_ (version 1.77+ on        | C++ libraries.                                        |
| Windows, 1.65+ otherwise)         |                                                       |
+-----------------------------------+-------------------------------------------------------+
| `Git`_                            | Command-line tool for retrieving source code.         |
+-----------------------------------+-------------------------------------------------------+
| `z3`_ (version 4.8+, Optional)    | For use with SMT checker.                             |
+-----------------------------------+-------------------------------------------------------+
| `cvc4`_ (Optional)                | For use with SMT checker.                             |
+-----------------------------------+-------------------------------------------------------+

.. _cvc4: https://cvc4.cs.stanford.edu/web/
.. _Git: https://git-scm.com/download
.. _Boost: https://www.boost.org
.. _CMake: https://cmake.org/download/
.. _z3: https://github.com/Z3Prover/z3

.. note::
    Solidity versions prior to 0.5.10 can fail to correctly link against Boost versions 1.70+.
    A possible workaround is to temporarily rename ``<Boost install path>/lib/cmake/Boost-1.70.0``
    prior to running the cmake command to configure solidity.

    Starting from 0.5.10 linking against Boost 1.70+ should work without manual intervention.

.. note::
    The default build configuration requires a specific Z3 version (the latest one at the time the
    code was last updated). Changes introduced between Z3 releases often result in slightly different
    (but still valid) results being returned. Our SMT tests do not account for these differences and
    will likely fail with a different version than the one they were written for. This does not mean
    that a build using a different version is faulty. If you pass ``-DSTRICT_Z3_VERSION=OFF`` option
    to CMake, you can build with any version that satisfies the requirement given in the table above.
    If you do this, however, please remember to pass the ``--no-smt`` option to ``scripts/tests.sh``
    to skip the SMT tests.

Minimum Compiler Versions
^^^^^^^^^^^^^^^^^^^^^^^^^

The following C++ compilers and their minimum versions can build the Solidity codebase:

- `GCC <https://gcc.gnu.org>`_, version 8+
- `Clang <https://clang.llvm.org/>`_, version 7+
- `MSVC <https://visualstudio.microsoft.com/vs/>`_, version 2019+

Prerequisites - macOS
---------------------

For macOS builds, ensure that you have the latest version of
`Xcode installed <https://developer.apple.com/xcode/download/>`_.
This contains the `Clang C++ compiler <https://en.wikipedia.org/wiki/Clang>`_, the
`Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_ and other Apple development
tools that are required for building C++ applications on OS X.
If you are installing Xcode for the first time, or have just installed a new
version then you will need to agree to the license before you can do
command-line builds:

.. code-block:: bash

    sudo xcodebuild -license accept

Our OS X build script uses `the Homebrew <https://brew.sh>`_
package manager for installing external dependencies.
Here's how to `uninstall Homebrew
<https://docs.brew.sh/FAQ#how-do-i-uninstall-homebrew>`_,
if you ever want to start again from scratch.

Prerequisites - Windows
-----------------------

You need to install the following dependencies for Windows builds of Solidity:

+-----------------------------------+-------------------------------------------------------+
| Software                          | Notes                                                 |
+===================================+=======================================================+
| `Visual Studio 2019 Build Tools`_ | C++ compiler                                          |
+-----------------------------------+-------------------------------------------------------+
| `Visual Studio 2019`_  (Optional) | C++ compiler and dev environment.                     |
+-----------------------------------+-------------------------------------------------------+
| `Boost`_ (version 1.77+)          | C++ libraries.                                        |
+-----------------------------------+-------------------------------------------------------+

If you already have one IDE and only need the compiler and libraries,
you could install Visual Studio 2019 Build Tools.

Visual Studio 2019 provides both IDE and necessary compiler and libraries.
So if you have not got an IDE and prefer to develop Solidity, Visual Studio 2019
may be a choice for you to get everything setup easily.

Here is the list of components that should be installed
in Visual Studio 2019 Build Tools or Visual Studio 2019:

* Visual Studio C++ core features
* VC++ 2019 v141 toolset (x86,x64)
* Windows Universal CRT SDK
* Windows 8.1 SDK
* C++/CLI support

.. _Visual Studio 2019: https://www.visualstudio.com/vs/
.. _Visual Studio 2019 Build Tools: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2019

We have a helper script which you can use to install all required external dependencies:

.. code-block:: bat

    scripts\install_deps.ps1

This will install ``boost`` and ``cmake`` to the ``deps`` subdirectory.

Clone the Repository
--------------------

To clone the source code, execute the following command:

.. code-block:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

If you want to help developing Solidity,
you should fork Solidity and add your personal fork as a second remote:

.. code-block:: bash

    git remote add personal git@github.com:[username]/solidity.git

.. note::
    This method will result in a prerelease build leading to e.g. a flag
    being set in each bytecode produced by such a compiler.
    If you want to re-build a released Solidity compiler, then
    please use the source tarball on the github release page:

    https://github.com/ethereum/solidity/releases/download/v0.X.Y/solidity_0.X.Y.tar.gz

    (not the "Source code" provided by github).

Command-Line Build
------------------

**Be sure to install External Dependencies (see above) before build.**

Solidity project uses CMake to configure the build.
You might want to install `ccache`_ to speed up repeated builds.
CMake will pick it up automatically.
Building Solidity is quite similar on Linux, macOS and other Unices:

.. _ccache: https://ccache.dev/

.. code-block:: bash

    mkdir build
    cd build
    cmake .. && make

or even easier on Linux and macOS, you can run:

.. code-block:: bash

    #note: this will install binaries solc and soltest at usr/local/bin
    ./scripts/build.sh

.. warning::

    BSD builds should work, but are untested by the Solidity team.

And for Windows:

.. code-block:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 16 2019" ..

In case you want to use the version of boost installed by ``scripts\install_deps.ps1``, you will
additionally need to pass ``-DBoost_DIR="deps\boost\lib\cmake\Boost-*"`` and ``-DCMAKE_MSVC_RUNTIME_LIBRARY=MultiThreaded``
as arguments to the call to ``cmake``.

This should result in the creation of **solidity.sln** in that build directory.
Double-clicking on that file should result in Visual Studio firing up.  We suggest building
**Release** configuration, but all others work.

Alternatively, you can build for Windows on the command-line, like so:

.. code-block:: bash

    cmake --build . --config Release

CMake Options
=============

If you are interested what CMake options are available run ``cmake .. -LH``.

.. _smt_solvers_build:

SMT Solvers
-----------
Solidity can be built against SMT solvers and will do so by default if
they are found in the system. Each solver can be disabled by a `cmake` option.

*Note: In some cases, this can also be a potential workaround for build failures.*


Inside the build folder you can disable them, since they are enabled by default:

.. code-block:: bash

    # disables only Z3 SMT Solver.
    cmake .. -DUSE_Z3=OFF

    # disables only CVC4 SMT Solver.
    cmake .. -DUSE_CVC4=OFF

    # disables both Z3 and CVC4
    cmake .. -DUSE_CVC4=OFF -DUSE_Z3=OFF

The Version String in Detail
============================

The Solidity version string contains four parts:

- the version number
- pre-release tag, usually set to ``develop.YYYY.MM.DD`` or ``nightly.YYYY.MM.DD``
- commit in the format of ``commit.GITHASH``
- platform, which has an arbitrary number of items, containing details about the platform and compiler

If there are local modifications, the commit will be postfixed with ``.mod``.

These parts are combined as required by SemVer, where the Solidity pre-release tag equals to the SemVer pre-release
and the Solidity commit and platform combined make up the SemVer build metadata.

A release example: ``0.4.8+commit.60cc1668.Emscripten.clang``.

A pre-release example: ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

Important Information About Versioning
======================================

After a release is made, the patch version level is bumped, because we assume that only
patch level changes follow. When changes are merged, the version should be bumped according
to SemVer and the severity of the change. Finally, a release is always made with the version
of the current nightly build, but without the ``prerelease`` specifier.

Example:

1. The 0.4.0 release is made.
2. The nightly build has a version of 0.4.1 from now on.
3. Non-breaking changes are introduced --> no change in version.
4. A breaking change is introduced --> version is bumped to 0.5.0.
5. The 0.5.0 release is made.

This behaviour works well with the  :ref:`version pragma <version_pragma>`.
