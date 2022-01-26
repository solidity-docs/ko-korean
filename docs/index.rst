솔리디티
========

솔리디티는 스마트 컨트랙트를 시도하기 위한 객체지향적인 고급 프로그래밍 언어입니다. 
여기서 스마트 컨트랙트란 이더리움 상태 내부에 있는 여러 계정들의 행동들을 통제할 수 있는 프로그램을 의미합니다.

솔리디티는 `curly-bracket language <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_.
C++, 파이썬 및 자바스크립트의 영향을 받았으며 이더리움 가상 머신(EVM)을 공략하기 위해 고안되었습니다.
솔리디티가 어떤 언어로부터 영감을 받았는지에 대한 자세한 설명은 :doc:`언어의 영향 <language-influences>` 섹션에서 확인하실 수 있습니다.

솔리디티는 정적 타입 언어로서 다른 특징 중에서도 상속, 라이브러리 그리고 유저가 스스로 정의한 복잡한 타입에 대한 정의 등을 지원합니다. 

솔리디티로 투표, 크라우드펀딩, 익명 경매 그리고 다중 서명 지갑에 사용되는 컨트랙트들을 만들 수 있습니다. 

컨트랙트를 배포할 시, 반드시 최신 버전의 솔리디티를 사용하시기 바랍니다. 예외적인 경우를 제외하곤 오로지 최신 버전만이 `보안 사항 픽스 <https://github.com/ethereum/solidity/security/policy#supported-versions>`_를 인정받기 때문입니다.
또한, 주기적으로 새로운 기능들이나 확연히 달라진 점들 소개될 예정입니다.  `빠르게 업데이트 되고 있음을 명시 <https://semver.org/#spec-item-4>`_하기 위하여 현재 저희 솔리디티 팀은 0.y.z 형식의 버전 넘버링 규칙을 사용하고 있습니다.

.. 주의::

  최근 솔리디티는 0.8.x 버전을 출시하면서 굉장히 많은 변동이 있었습니다. 
  반드시 :doc:`전체 리스트 <080-breaking-changes>`를 확인하시기 바랍니다.

솔리디티 혹은 공식 문서의 보다 나은 발전은 언제든지 환영합니다. 
자세한 사항은 :doc:`기여자 가이드 <contributing>`를 확인해 주십시오.

.. 힌트::

  해당 공식 문서는 좌측 하단에 있는 버전 드롭다운 메뉴를 클릭하여 PDF, HTML 혹은 Epub와 같이 원하는 다운로드 형식으로 다운로드 하실 수 있습니다. 


시작하기
---------------

**1. 스마트 컨트랙트의 기초 이해하기**

만약 여러분이 스마트 컨트랙트가 처음이라면 "스마트 컨트랙트 개요" 섹션부터 시작하는 것을 추천드립니다. 해당 섹션은 다음 파트들을 다루고 있습니다. 

* 솔리디티로 구성된 :ref:`간단한 스마트 컨트랙트의 예시 <simple-smart-contract>`
* :ref:`블록체인의 기초 <blockchain-basics>`.
* :ref:`이더리움 가상 머신 <the-ethereum-virtual-machine>`.

**2. Get to Know Solidity**

Once you are accustomed to the basics, we recommend you read the :doc:`"Solidity by Example" <solidity-by-example>`
and “Language Description” sections to understand the core concepts of the language.

**3. Install the Solidity Compiler**

There are various ways to install the Solidity compiler,
simply choose your preferred option and follow the steps outlined on the :ref:`installation page <installing-solidity>`.

.. hint::
  You can try out code examples directly in your browser with the
  `Remix IDE <https://remix.ethereum.org>`_. Remix is a web browser based IDE
  that allows you to write, deploy and administer Solidity smart contracts, without
  the need to install Solidity locally.

.. warning::
    As humans write software, it can have bugs. You should follow established
    software development best-practices when writing your smart contracts. This
    includes code review, testing, audits, and correctness proofs. Smart contract
    users are sometimes more confident with code than their authors, and
    blockchains and smart contracts have their own unique issues to
    watch out for, so before working on production code, make sure you read the
    :ref:`security_considerations` section.

**4. Learn More**

If you want to learn more about building decentralized applications on Ethereum, the
`Ethereum Developer Resources <https://ethereum.org/en/developers/>`_
can help you with further general documentation around Ethereum, and a wide selection of tutorials,
tools and development frameworks.

If you have any questions, you can try searching for answers or asking on the
`Ethereum StackExchange <https://ethereum.stackexchange.com/>`_, or
our `Gitter channel <https://gitter.im/ethereum/solidity/>`_.

.. _translations:

Translations
------------

Community volunteers help translate this documentation into several languages.
They have varying degrees of completeness and up-to-dateness. The English
version stands as a reference.

.. note::

   We recently set up a new GitHub organization and translation workflow to help streamline the
   community efforts. Please refer to the `translation guide <https://github.com/solidity-docs/translation-guide>`_
   for information on how to contribute to the community translations moving forward.

* `French <https://solidity-fr.readthedocs.io>`_ (in progress)
* `Italian <https://github.com/damianoazzolini/solidity>`_ (in progress)
* `Japanese <https://solidity-jp.readthedocs.io>`_
* `Korean <https://solidity-kr.readthedocs.io>`_ (in progress)
* `Russian <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (rather outdated)
* `Simplified Chinese <https://learnblockchain.cn/docs/solidity/>`_ (in progress)
* `Spanish <https://solidity-es.readthedocs.io>`_
* `Turkish <https://github.com/denizozzgur/Solidity_TR/blob/master/README.md>`_ (partial)

Contents
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2
   :caption: Basics

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst

.. toctree::
   :maxdepth: 2
   :caption: Language Description

   layout-of-source-files.rst
   structure-of-a-contract.rst
   types.rst
   units-and-global-variables.rst
   control-structures.rst
   contracts.rst
   assembly.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: Compiler

   using-the-compiler.rst
   analysing-compilation-output.rst
   ir-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: Internals

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimizer.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Additional Material

   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   080-breaking-changes.rst
   natspec-format.rst
   security-considerations.rst
   smtchecker.rst
   resources.rst
   path-resolution.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   brand-guide.rst
   language-influences.rst
