솔리디티
========

솔리디티는 스마트 컨트랙트를 시도하기 위한 객체지향적인 고급 프로그래밍 언어입니다. 
여기서 스마트 컨트랙트란 이더리움 상태 내부에 있는 여러 계정들의 행동들을 통제할 수 있는 프로그램을 의미합니다.

<<<<<<< HEAD
솔리디티는 `curly-bracket language <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_ 입니다.
C++, 파이썬 및 자바스크립트의 영향을 받았으며 이더리움 가상 머신(EVM)을 공략하기 위해 고안되었습니다.
솔리디티가 어떤 언어로부터 영감을 받았는지에 대한 자세한 설명은 :doc:`language influeces <language-influences>` 섹션에서 확인하실 수 있습니다.
=======
Solidity is a `curly-bracket language <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_ designed to target the Ethereum Virtual Machine (EVM).
It is influenced by C++, Python and JavaScript. You can find more details about which languages Solidity has been inspired by in the :doc:`language influences <language-influences>` section.
>>>>>>> 056c4593e37bdbd929d0ef538462242c7ddcbf21

솔리디티는 정적 타입 언어로서 다른 특징 중에서도 상속, 라이브러리 그리고 유저가 스스로 정의한 복잡한 타입에 대한 정의 등을 지원합니다. 

솔리디티로 투표, 크라우드펀딩, 익명 경매 그리고 다중 서명 지갑에 사용되는 컨트랙트들을 만들 수 있습니다. 

컨트랙트를 배포할 시, 반드시 최신 버전의 솔리디티를 사용하시기 바랍니다. 예외적인 경우를 제외하곤 오로지 최신 버전만이 `security fixes <https://github.com/ethereum/solidity/security/policy#supported-versions>`_ 를 인정하기 때문입니다.
또한, 주기적으로 새로운 기능들이나 확연히 달라진 점들 소개될 예정입니다.  `빠르게 업데이트 되고 있음을 명시 <https://semver.org/#spec-item-4>`_ 하기 위하여 현재 저희 솔리디티 팀은 0.y.z 형식의 버전 넘버링 규칙을 사용하고 있습니다.

.. 주의::

  최근 솔리디티는 0.8.x 버전을 출시하면서 굉장히 많은 변동이 있었습니다. 
  반드시 :doc:`전체 리스트 <080-breaking-changes>` 를 확인하시기 바랍니다.

솔리디티 혹은 공식 문서의 보다 나은 발전은 언제든지 환영합니다. 
자세한 사항은 :doc:`contributors guide <contributing>` 를 확인해 주십시오.

.. 힌트::

  해당 공식 문서는 좌측 하단에 있는 버전 드롭다운 메뉴를 클릭하여 PDF, HTML 혹은 Epub와 같이 원하는 다운로드 형식으로 다운로드 하실 수 있습니다. 


시작하기
---------------

**1. 스마트 컨트랙트의 기초 이해하기**

스마트 컨트랙트가 처음이시라면 "스마트 컨트랙트 개요" 섹션부터 시작하는 것을 추천드립니다. 해당 섹션은 다음 파트들을 다루고 있습니다. 

* 솔리디티로 구성된 :ref:`간단한 스마트 컨트랙트의 예시 <simple-smart-contract>`
* :ref:`블록체인의 기초 <blockchain-basics>`.
* :ref:`이더리움 가상 머신 <the-ethereum-virtual-machine>`.

**2. Solidity에 대하여**

기초에 대해 조금 익숙해지셨다면 핵심 개념을 이해하기 위하여 :doc:`"예제로 살펴보는 Solidity" <solidity-by-example>` 및 "언어의 설명" 섹션 부분을 읽어보시길 바랍니다.

**3. Solidity 컴파일러 설치하기**

Solidity 컴파일러를 설치하는 방법은 다양합니다. 원하는 옵션을 선택하신 후 :ref:`installation page <installing-solidity>` 에 설명된 단계를 따르십시오. 

.. 힌트::
   `Remix IDE <https://remix.ethereum.org>`_ 를 통해 예시 코드를 브라우저에서 직접 살펴볼 수도 있습니다. 
   Remix는 Solidity를 로컬에서 설치할 필요 없이 Solidity 스마트 컨트랙트를 작성, 배포 및 관리할 수 있도록 도와주는 브라우저 기반의 IDE입니다.

.. 경고::
   소프트웨어도 결국 사람이 작성한 것이라 버그가 있을 수 있습니다.
   따라서 스마트 컨트랙트 작성 시 최선의 소프트웨어 개발법을 사용하시기 바랍니다.
   코드 리뷰, 테스팅, 코드 감사 그리고 변경 내역 표기 등이 대표적인 예입니다. 
   스마트 컨트랙트 사용자는 종종 작성자보다 코드에 대해 더욱 자신있어 합니다. 
   또한 블록체인과 스마트 컨트랙트 모두 고유의 이슈가 있을 수 있으므로 배포 코드를 작성하기 전에 :ref:`security_considerations` 섹션을 읽어보시기 바랍니다.

**4. 더 알아보기**

Ethereum 상에서 동작하는 탈중앙화 어플리케이션 제작과 관련하여 더 알고 싶으시다면 `Ethereum Developer Resources <https://ethereum.org/en/developers/>`_ 를 참조해 주시기 바랍니다.
Ethereum에 대한 더 많은 문서, 다양한 튜토리얼, 툴, 그리고 개발 프레임워크에 대한 내용이 포함되어 있습니다.

궁금한 점이 있으시다면 검색, `Ethereum StackExchange <https://ethereum.stackexchange.com/>`_ 혹은 저희 `Gitter channel <https://gitter.im/ethereum/solidity/>`_ 에 질문을 남겨주시기 바랍니다.

.. _translations:

번역
------------

<<<<<<< HEAD
현재 커뮤니티 자원자분들께서 본 공식 문서를 몇몇 언어로 번역하고 계십니다. 
완성도 및 업데이트 상황은 언어별로 상이하며, 영문이 원본입니다. 

.. 참조::

   번역 작업을 보다 수월하게 진행되도록 하기 위하여 저희 팀은 최근에 새로운 Github organization을 새로 생성하였습니다. 
   관심 있으신 분들은 `번역 가이드 <https://github.com/solidity-docs/translation-guide>`_ 부분을 참조하시어 어떻게 참여하실 수 있는지 알아보시기 바랍니다.

* `프랑스어 <https://solidity-fr.readthedocs.io>`_ (진행 중)
* `이탈리아어 <https://github.com/damianoazzolini/solidity>`_ (진행 중)
* `일본어 <https://solidity-jp.readthedocs.io>`_
* `한국어 <https://solidity-kr.readthedocs.io>`_ (진행 중)
* `러시아어 <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (업데이트 중단)
* `중국어(간체) <https://learnblockchain.cn/docs/solidity/>`_ (진행 중)
* `스페인어 <https://solidity-es.readthedocs.io>`_
* `터키어 <https://github.com/denizozzgur/Solidity_TR/blob/master/README.md>`_ (부분적으로 진행)
=======
Community contributors help translate this documentation into several languages.
Note that they have varying degrees of completeness and up-to-dateness. The English
version stands as a reference.

You can switch between languages by clicking on the flyout menu in the bottom-left corner
and selecting the preferred language.

* `Chinese <https://github.com/solidity-docs/zh-cn-chinese/>`_
* `French <https://docs.soliditylang.org/fr/latest/>`_
* `Indonesian <https://github.com/solidity-docs/id-indonesian>`_
* `Japanese <https://github.com/solidity-docs/ja-japanese>`_
* `Korean <https://github.com/solidity-docs/ko-korean>`_
* `Persian <https://github.com/solidity-docs/fa-persian>`_
* `Russian <https://github.com/solidity-docs/ru-russian>`_
* `Spanish <https://github.com/solidity-docs/es-spanish>`_
* `Turkish <https://github.com/solidity-docs/tr-turkish>`_

.. note::

   We set up a GitHub organization and translation workflow to help streamline the
   community efforts. Please refer to the translation guide in the `solidity-docs org <https://github.com/solidity-docs>`_
   for information on how to start a new language or contribute to the community translations.
>>>>>>> 056c4593e37bdbd929d0ef538462242c7ddcbf21

차례
========

:ref:`키워드 색인 <genindex>`, :ref:`검색 <search>`

.. toctree::
   :maxdepth: 2
   :caption: 기초

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
