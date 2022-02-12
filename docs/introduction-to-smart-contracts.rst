###############################
스마트 컨트랙트의 기초
###############################

.. _simple-smart-contract:

***********************
간단한 스마트 컨트랙트
***********************

변수값을 설정하고 이를 다른 컨트랙트에서 접근 가능하도록 노출시켜보는 간단한 예제를 만드는 것부터 시작해보겠습니다.
지금 당장 이해가 안되더라도 걱정 마십시오. 앞으로 더 자세한 내용을 다룰 예정입니다.

Storage 예제
===============

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract SimpleStorage {
        uint storedData;

        function set(uint x) public {
            storedData = x;
        }

        function get() public view returns (uint) {
            return storedData;
        }
    }

첫번째 줄은 해당 소스 코드가 GPL 3.0 버전 라이센스 기준으로 작성되었음을 알립니다.
기계가 읽을 수 있는 라이센스 표시자는 소스 코드를 기본값인 설정에서 중요한 역할을 합니다.

다음 줄은 Solidity 0.4.16 버전으로 작성되었거나 0.9.0 버전은 포함되지 않은 새 버전의 언어로 작성되었음을 표시합니다.
이는 해당 컨트랙트가 새 (오류가 날 수 있는) 컴파일러 버전에서는 호환이 되지 않아 다르게 동작할 수 있음을 명시하는 역할을 합니다.
:ref:`Pragmas<pragma>` 는 컴파일러에게 어떻게 소스 코드를 다뤄야 하는지 알려주는 일반적인 설명서를 의미합니다 (예: `pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_)

Solidity 내에서의 컨트랙트는 Ethereum 블록체인 상의 특정 주소에 있는 코드(*함수*)와 데이터(*상태*)의 집합체라 볼 수 있습니다.
코드 ``uint storedData;`` 는 ``uint`` (256비트의 무부호 정수) 타입의 ``storedData`` 라는 상태 변수를 선언하고 있습니다.
데이터베이스를 다루는 함수 코드들을 조회 및 수정할 수 있는 일종의 데이터베이스 내부의 단일 슬롯이라 생각하시면 됩니다.
예제에선 컨트랙트가 변수의 값을 바꾸거나 조회해올 수 있는 ``set`` 과 ``get`` 함수를 정의하고 있습니다. 

현재 컨트랙트에서 (상태 변수와 같은) 멤버에 접근하기 위해서 굳이 ``this.`` 접두어를 쓸 필요 없이 이름에 직접 접근하실 수 있습니다.
다른 언어와는 다르게 이를 생략하는 것은 스타일적인 것 뿐만이 아니라 멤버에 접근하는 완전히 다른 방식이기도 하지만 이는 추후에 살펴보도록 하겠습니다.

이 컨트랙트는 (Ethereum에 기반해 있기 때문에) 전 세계의 누구나 접근할 수 있는 단순 숫자를 여러분이 배포하는 것을 박는 (실질적인) 방법 없이 모두가 저장할 수 있도록 해주는 것 이외에 별다른 역할을 하고 있진 않습니다.
누구든지 다른 값을 ``set`` 을 호출함으로서 여러분의 숫자를 덮어씌울 수 있지만, 해당 숫자는 여전히 블록체인의 히스토리에 저장되어 있습니다.
추후에 어떻게 하면 여러분만이 숫자를 변경할 수 있도록 제한을 걸 수 있는지에 대해 알아보도록 하겠습니다.

.. 경고::
    유니코드 텍스트를 사용하실 땐 주의하십시오.
    비슷하거나 똑같은 철자라도 엄연히 다른 코드 포인트를 가지고 있어 다른 바이트 배열로 인코딩될 수 있습니다.

.. 참조::
    모든 식별자(컨트랙트 이름, 함수 이름 및 변수 이름)는 ASCII 캐릭터 세트를 엄수합니다.
    UTF-8 인코딩 데이터는 문자열 변수로 저장이 가능합니다.

.. index:: ! subcurrency

Subcurrency 예제 
===================

아래 컨트랙트는 가상화폐의 가장 간단한 폼을 시행합니다. 
해당 컨트랙트는 오직 생성자만 새로운 코인(다른 scheme을 발행하는 것도 가능)을 생성하게끔 해줍니다. 
코인을 서로 주고받기 위해 별도로 아이디나 비밀번호를 등록할 필요 없이 Ethereum keypair만 있으면 됩니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract Coin {
        // "public" 키워드는 변수를
        // 다른 컨트랙트로부터 접근 가능하게 하도록 해줍니다.
        address public minter;
        mapping (address => uint) public balances;

        // Event는 클라이언트에게 여러분이 선언한
        // 특정 컨트랙트의 변화에 반응할 수 있도록 해줍니다.
        event Sent(address from, address to, uint amount);

        // Constructor 코드는 오직 
        // 컨트랙트가 생성될 때에만 시행됩니다.
        constructor() {
            minter = msg.sender;
        }

        // 주소로 새롭게 생성된 코인의 일정량을 전송합니다.
        // 오직 컨트랙트의 생성자에 의해서만 호출될 수 있습니다.
        function mint(address receiver, uint amount) public {
            require(msg.sender == minter);
            balances[receiver] += amount;
        }

        // Error는 해당 작업이 왜 실패했는지에 대한 정보를 제공해줍니다. 
        // Error는 함수의 호출자에게 반환됩니다. 
        error InsufficientBalance(uint requested, uint available);

        // 호출자 누구에게서든 기존에 갖고 있던
        // 일정량의 코인을 주소로 전송합니다. 
        function send(address receiver, uint amount) public {
            if (amount > balances[msg.sender])
                revert InsufficientBalance({
                    requested: amount,
                    available: balances[msg.sender]
                });

            balances[msg.sender] -= amount;
            balances[receiver] += amount;
            emit Sent(msg.sender, receiver, amount);
        }
    }

이 컨트랙트는 새로운 개념을 소개하고 있습니다. 하나씩 살펴보도록 하겠습니다.

``address public minter;`` 코드는 :ref:`address<address>` 타입의 상태 변수를 선언하고 있습니다.
``address`` 타입은 어떠한 기하학적 연산도 허용하지 않는 160 비트의 값입니다.
이는 컨트랙트의 주소 혹은 :ref:`외부 계정<accounts>` 에 속해있는 반쪽의 공개 keypair 해시값을 저장할 때 유용합니다. 

``public`` 키워드는 컨트랙트 외부에 있는 상태 변수의 현재값에 접근할 수 있도록 도와주는 함수를 자동적으로 생성해줍니다.
이 키워드가 없이는 다른 컨트랙트는 해당 변수에 접근할 수 없습니다. 
컴파일러에 의해 생성되는 함수 코드는 다음과 같습니다 (지금은 ``external`` 과 ``view`` 를 무시해주십시오.)

.. code-block:: solidity

    function minter() external view returns (address) { return minter; }

여러분 스스로 위와 같이 함수를 추가해도 되지만 이러면 상태 변수와 함수의 이름이 동일해집니다.
컴파일러가 알아서 해결해주니 이렇게 하실 필요가 없습니다. 

.. index:: mapping

다음 줄의 ``mapping (address => uint) public balances;`` 또한 public 상태 변수를 생성하지만 이번에는 더욱 복잡한 데이터 타입입니다.
:ref:`mapping <mapping-types>` 타입은 :ref:`무부호 정수 <integers>` 로 주소를 매핑합니다.

매핑은 `해시 테이블 <https://en.wikipedia.org/wiki/Hash_table>`_ 과 같이 가상으로 초기화되어 더욱 많은 모든 키들이 시작부터 존재하며
모두 0의 바이트로 표시된 값으로 매핑됩니다. 
하지만, 매핑의 모든 키 리스트 혹은 모든 값 리스트를 가져오는 것을 불가능합니다.
매핑을 하시면서 여러분이 어떤 것을 추가하였는지 기록하거나 이것이 필요 없는 컨텍스트에 사용하십시오.
더욱 좋은 방법은 리스트로 저장해두거나 알맞은 데이터 타입을 사용하는 것입니다.

``public`` 키워드로 생성된 :ref:`getter 함수<getter-functions>` 는 아래와 같이 매핑할 때 더욱 복잡해집니다.

.. code-block:: solidity

    function balances(address _account) external view returns (uint) {
        return balances[_account];
    }

해당 함수를 개인 계정에 남아 있는 잔액을 조회할 때 사용할 수 있습니다.

.. index:: event

코드 ``event Sent(address from, address to, uint amount);`` 는 :ref:`"event" <events>` 를 선언하는데, 이는 마지막 줄의 ``send`` 함수로부터 발생됩니다.
이렇게 웹 어플리케이션과 같은 Ethereum 클라이언트는 큰 비용을 지불하지 않고도 블록체인 내부의 event들을 주시할 수 있습니다.
event가 발생할 경우 listener는 트랜잭션을 추적할 수 있도록 도와주는 ``from``, ``to`` 그리고 ``amount`` 인수를 받게 됩니다.

event를 주시하기 위해 아래 ``Coin`` 컨트랙트 객체를 만들기 위해 사용되는 `web3.js <https://github.com/ethereum/web3.js/>`_ JavaScript 코드를 사용해보실 수도 있습니다.
그리고 모든 유저 인터페이스는 위에서 자동적으로 생성된 ``balances`` 함수를 호출합니다. 

    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Coin transfer: " + result.args.amount +
                " coins were sent from " + result.args.from +
                " to " + result.args.to + ".");
            console.log("Balances now:\n" +
                "Sender: " + Coin.balances.call(result.args.from) +
                "Receiver: " + Coin.balances.call(result.args.to));
        }
    })

.. index:: coin

:ref:`constructor <constructor>` 는 컨트랙트 생성 시에만 실행되고 그 뒤로는 실행되지 않는 특수 함수입니다. 예제에선 컨트랙트를 생성하는 사람의 주소를 영구히 저장합니다.
(``tx`` 와 ``block`` 처럼) ``msg`` 변수는 :ref:`특수 전역 변수 <special-variables-functions>` 로써, 블록체인에 접근할 수 있는 프로퍼티들을 가지고 있습니다.
``msg.sender`` 는 항시 현재 (외부에서) 호출되는 함수의 주소가 됩니다.  

함수가 컨트랙트를 생성하고, 이에 따라 유저와 컨트랙트가 호출할 수 있는 함수는 ``mint`` 와 ``send`` 입니다.

``mint`` 함수는 새로 생성된 코인을 다른 주소로 보내줍니다. :ref:`require <assert-and-require>` 함수는 모든 변경 사항을 취소하는 조건을 정의합니다.
예제의 ``require(msg.sender == minter);`` 부분이 컨트랙트 생성자만 ``mint`` 함수를 호출할 수 있음을 보장하게 합니다.
보통, 생성자는 원하는 만큼 토큰을 만들 수 있지만, 이럴 경우 가끔 "overflow"라 하는 현상을 초래할 수 있습니다. 
기본적인 :ref:`Checked arithmetic <unchecked>` 으로 인해, ``balances[receiver] += amount;`` 부분에서 overflow가 발생할 경우 
(즉 ``balances[receiver] + amount`` 부분의 arbitrary precision arithmetic이 ``uint`` (``2**256 - 1``)의 최댓값보다 클 경우) 트랜잭션은 되돌아갑니다.

:ref:`Errors <errors>` 는 호출자에게 조건 혹은 처리 과정이 왜 실패했는지에 대한 정보를 제공해줍니다.
Error는 :ref:`revert statement <revert-statement>` 와 함께 사용됩니다. 
revert statement는 무조건적으로 종료하고 ``require`` 함수와 비슷하게 모든 변경 사항들을 원상복귀시킵니다. 
다만, 동시에 호출자(궁극적으로는 프론트엔드 어플리케이션 혹은 블록 탐색자)에게 전달될 오류 이름과 추가적인 데이터를 제공하기도 합니다. 
따라서 실패를 쉽게 디버깅하거나 조기에 발견할 수 있게 됩니다. 

``send`` 함수는 (가지고 있는 코인을) 어느 누구에게든지 보내고자 하는 모든 사람들에 의해 사용될 수 있습니다. 
만일 전송자가 전송하고자 하는 코인의 양이 충분치 않을 경우, ``if`` 조건은 참으로 판별하게 됩니다. 
결과적으로, ``revert`` 가 작업을 실패로 처리하면서 전송자에게 ``InsufficientBalance`` 에러를 통해 에러의 세부사항들을 알려줍니다.

.. 참조::
    위 컨트랙트를 사용해 주소로 코인을 전송할 때 블록체인 탐색자에 나타나는 주소 상에는 아무 것도 보이지 않을겁니다.
    이는 여러분들께서 코인을 전송했다는 기록과 변경된 잔액은 특정 코인 컨트랙트의 데이터 스토리지에만 저장되기 때문입니다. 
    event를 사용함으로서 여러분은 트랜잭션과 새로운 코인의 잔액을 추적하는 "블록체인 탐색자"를 생성할 수 있게 됩니다. 
    하지만, 이 경우 코인 소유자의 주소가 아닌 코인 컨트랙트의 주소를 살펴보셔야 합니다.

.. _blockchain-basics:

*****************
블록체인의 기초
*****************

프로그래머에겐 블록체인의 개념이 아주 어렵게 다가오진 않을겁니다. 
이유는 대부분의 복잡한 개념들 (채굴, `해싱 <https://en.wikipedia.org/wiki/Cryptographic_hash_function>`_, `타원곡선 암호 <https://en.wikipedia.org/wiki/Elliptic_curve_cryptography>`_, `동등 계층 통신망 <https://en.wikipedia.org/wiki/Peer-to-peer>`_ 등)
은 단순히 플랫폼을 위한 특징과 약속을 설명하기 위한 도구일 뿐, 여러분들이 해당 특징들만 이해하게 된다면 그 밑바닥의 기술에 대해서는 걱정하지 않으셔도 됩니다. 
마치 아마존의 AWS를 쓰기 위해 내부까지 전부 알아야 될 필요가 없듯이요. 

.. index:: transaction

Transactions
============

블록체인은 전 세계적으로 공유되는 거래 기반의 데이터베이스입니다.
이는 네트워크에만 접속하기만 하면 누구든지 데이터베이스 엔트리를 읽을 수 있다는 의미입니다. 
만일 데이터베이스에 무언가를 바꾸고 싶다면, 여러분은 타인도 인정할 수 있는 소위 트랜잭션이라는 것을 생성해야 합니다. 
트랜잭션이란 용어는 여러분이 바꾸려는 사항(예컨대 두 값을 동시에 바꾼다는 등)은 아예 이루어지지 않거나 혹은 완전히 적용될 수 있음을 암시합니다. 
나아가, 여러분의 트랜잭션이 데이터베이스에 적용되게 되면, 어떤 다른 트랜잭션도 그것을 변경할 수 없습니다. 

예를 들어, 전자 통화로 표시된 모든 계좌의 잔액 리스트를 보여주는 표가 있다고 가정해 보겠습니다. 
한 계좌에서 다른 계좌로의 이체 요청이 발생하면, 데이터베이스의 기본적인 거래 성질에 따라 한 계좌에서 특정 양이 감소가 되면 다른 한 쪽은 항상 그마만큼 추가가 된다는 것을 의미합니다. 
어떠한 이유든지 간에 만일 상대방 계좌 상에서 해당 양만큼 증가가 이루어지지 않는다면 이는 원래 계좌에서 또한 감소가 이루어지지 않게 됩니다. 

또한, 트랜잭션은 항상 전송자(생성자)에 의해 암호화된 서명을 받게 됩니다. 
이렇게 함으로서 데이터베이스의 특정 변경에 대한 접근을 직접적으로 보호할 수 있게 됩니다. 
전자 통화 예제에서 볼 수 있듯이, 계좌의 키를 가지고 있는 오직 한 사람만이 돈을 이체할 수 있습니다. 

.. index:: ! block

블록
======

(Bitcoin 용어로) "double-spend attack"라 하는 큰 문제점이 있습니다. 
만일 한 네트워크 안에 계좌를 동시에 비우고 싶어하는 두 개의 서로 다른 트랜잭션이 발생한다면 어떻게 될까요? 
평상적으로 맨 첫번째로 인정되는 트랜잭션만이 유효하게 될겁니다. 문제는 peer-to-peer network 상에서 "첫번째"라는 단어가 그리 객관적인 용어가 아니라는 점에 있습니다.  

간단히 말씀드리자면 이 부분에 대해 크게 신경쓰실 필요가 없습니다. 여러분들께 널리 통용되는 트랜잭션의 순서가 주어져서 문제를 해결하기 때문입니다.
트랜잭션은 "블록"이라는 것으로 묶여지며 참가하는 모든 노드에게 전파되고 실행됩니다. 
만일 두 개의 서로 다른 트랜잭션이 충돌을 일으킬 경우, 두 번째로 오는 트랜잭션은 거절되며 블록의 한 부분이 되지 못합니다. 

이 블록들은 시간의 선형 시퀀스를 형성하며, 이것이 바로 "블록체인"이라는 용어가 탄생하게 된 계기입니다. 
블록들은 일정한 간격으로 체인에 추가가되며, Ethereum의 경우 대략 매 17초가 걸립니다. 

순서 선택 메카니즘("채굴"이라고도 부릅니다)의 한 부분으로써 블록들은 시간에 따라 회귀하지만 오직 체인의 끝부분에서만 일어납니다. 
특정 블록의 상단에 블록들이 추가되면 될수록 회귀되는 확률은 적어집니다. 따라서, 여러분의 트랜잭션들이 회귀될 수 있으며 블록체인에서 제거된다 하더라도 더욱 오래 기다릴수록 그럴 확률이 적어지게 됩니다.

.. 참고::
    트랜잭션들이 다음 블록이나 혹은 미래의 어떠한 블록에 항상 추가가 된다고 보장될 순 없습니다. 
    왜냐하면 이는 트랜잭션의 제출자가 아닌 어떤 블록에 트랜잭션을 포함시킬지 결정하는 채굴자에 달려 있기 때문입니다.

    만일 여러분이 만드신 컨트랙트의 미래 호출을 스케쥴링하고 싶으시다면, 스마트 컨트랙트 자동화 툴이나 오라클 서비스를 이용하실 수 있습니다.

.. _the-ethereum-virtual-machine:

.. index:: !evm, ! ethereum virtual machine

****************************
Ethereum 가상 머신
****************************

개요
========

Ethereum 가상 머신 (혹은 EVM)은 Ethereum 내의 스마트 컨트랙트를 위한 런타임 환경입니다. 
보호된 영역에서 실행될 뿐만 아니라 완전히 독립적이기 때문에 EVM 내부에서 동작하는 코드들은 네트워크, 파일 시스템 혹은 기타 프로세스에 접근할 수 없습니다. 
스마트 컨트랙트는 심지어 다른 스마트 컨트랙트에 제한적으로 접근할 수 밖에 없습니다.

.. index:: ! account, address, storage, balance

.. _accounts:

계정
========

Ethereum에는 동일한 주소를 공유하고 있는 두 가지 종류의 계정이 있습니다. 
그 중 하나는 개인-공개 키쌍(즉, 인간)에 의해 관리되는 **외부 계정**이며, 다른 하나는 계정과 함께 코드에 의해 관리되는 **컨트랙트 계정**입니다.

외부 계정의 주소는 공개키에 의해 결정되며, 컨트랙트의 주소는 컨트랙트가 생성되는 시점에서 결정됩니다. 
이는 생성자의 주소와 소위 "nonce"라 하는 해당 주소로부터 전송된 트랜잭션의 수에서 비롯됩니다.

게정이 코드를 저장하는지의 유무와는 상관없이, 두 가지 계정 모두 EVM에 의해 동등하게 취급됩니다.

모든 계정은 **storage**라 불리는 256 비트의 글자에서 256 비트의 글자로 매핑하는 일관된 키-값 저장된 값을 가지고 있습니다.

또한, 모든 계정은 Ehter(``1 ether`` 는 ``10**18 wei`` 입니다) 단위로 표시된 **balance**가 있으며 이는 Ether를 포함하는 트랜잭션을 전송함으로서 바뀝니다.

.. index:: ! transaction

트랜잭션
============

트랜잭션이란 한 계정에서 다른 계정으로 전송되는 메세지(같거나 혹은 빈 형태일 수도 있습니다. 아래를 참조해주세요)입니다.
트랜잭션은 binary 데이터("payload"라 불립니다) 및 Ether를 포함할 수 있습니다.

타겟 계정이 코드를 포함하고 있으면, 해당 코드가 실행되며 payload는 입력 데이터로써 제공됩니다.

만일 타겟 게정이 설정되어 있지 않다면 (트랜잭션이 수취인이 없거나 수취인이 ``null`` 로 설정되어 있다면), 트랜잭션은 **새로운 컨트랙트**를 생성합니다.
이미 언급 드렸다시피, 컨트랙트의 주소는 zero 주소가 아니라 전송자와 ("nonce"를 보낸) 트랜잭션의 수로부터 비롯된 주소입니다. 
컨트랙트를 생성하는 이러한 트랜잭션의 payload는 EVM의 바이트 코드로 전송되어 실행됩니다. 
이 실행에 따른 출력 데이터는 컨트랙트의 코드로써 영구히 저장됩니다. 
이는 컨트랙트를 생성하기 위해선 컨트랙트의 실제 코드를 전송하는 것이 아니라 실제로는 실행될 때 해당 코드를 반환하는 코드를 전송하는 것을 의미합니다.

.. 참조::
    컨트랙트가 생성되는 동안 여전히 해당 코드는 비어 있습니다. 
    따라서, 여러분은 constructor가 실행을 끝날 때까지 제작되고 있는 컨트랙트를 콜백해서는 안 됩니다. 

.. index:: ! gas, ! gas price

가스
===

생성이 되고 난 후, 각 트랜잭션은 일정량의 **가스**를 지불하게 됩니다. 
이는 트랜잭션을 실행하기 위해 필요한 작업량을 제한함과 동시에 작업을 수행하기 위하여 지불하기 위함입니다. 
EVM이 트랜잭션을 실행하는 동안, 가스는 특정 규칙에 따라 점차적으로 고갈됩니다.

**가스 가격**은 전송 계좌로부터 미리 ``gas_price * gas`` 양만큼 지불해야 하는 트랜잭션 생성자가 설정한 값입니다. 
만일 실행 이후 약간의 가스가 남게 된다면, 그 가스는 똑같이 생성자에게 환불됩니다.

만일 가스가 일정 수준까지 사용하게 될 경우 (즉 음수가 될 경우), out-of-gas 예외가 발생되며 현재 프레임의 상태에 맞춰 모든 변경 사항이 취소가 됩니다. 

.. index:: ! storage, ! memory, ! stack

스토리지, 메모리 및 스택
=============================

Ethereum 가상 머신은 데이터를 저장할 수 있는 세 가지 공간이 있는데, 바로 스토리지, 메모리 그리고 스택입니다.
다음 단락에서 바로 설명드리도록 하겠습니다.

각 계정은 **스토리지**라는 데이터 공간이 있는데 함수의 호출과 트랜잭션 사이에서 지속적으로 존재합니다. 
스토리지는 256 비트 단어를 256 비트의 단어로 매핑해주는 키-값 저장소입니다. 
컨트랙트로부터 스토리지를 열거하는 것은 불가능합니다. 스토리지를 읽는 것은 상대적으로 값비싸며 초기화하거나 변경을 할 경우 더욱 비싸집니다. 
이러한 비용 때문에 여러분들은 지속적 스토리지 안에 저장할 것부터 컨트랙트를 실행시키기 위해 필요한 것까지를 최소화해야 합니다. 
도출된 계산, 캐싱 그리고 aggregate 같은 데이터들은 컨트랙트 외부에 저장하십시오. 
컨트랙트는 자기 자신 이외에 어떠한 스토리지에 읽거나 쓸 수 없습니다. 

두번째 데이터 영역은 새롭게 정리된 각 메세지 콜에 대한 인스턴스를 가지는 컨트랙트인 **메모리**입니다. 
메모리는 선형적이며 바이트 레벨로 주소화되지만 읽기에는 256비트까지만 허용되며 쓰기에는 8비트에서 256비트 사이에서 가능합니다. 
메모리는 (읽거나 쓰기를 통해) 이전에 접촉되지 않은 메모리 글자(예를 들어 글자 안의 모든 상쇄)에 접근할 때 워드에 의해 확장됩니다. 
확장 시, 반드시 가스가 지불됩니다. 메모리는 크기가 커질수록 비용 또한 증가합니다 (2차식으로 증가합니다).

EVM은 register machine이 아닌 stack machine으로써, 모든 계산은 **스택**이라 불리는 데이터 영역에서 행해집니다. 
스택은 최대 1024개의 요소들과 256비트의 단어들로 구성됩니다. 스택에 접근하는 것은 최상단부터만 가능하며 다음과 같은 방법으로 진행됩니다:
최상단의 16개 요소들 중 하나를 스택의 상단에 복사를 하거나 혹은 최상단의 요소를 16개 요소들 중 하나의 밑으로 바꿉니다. 
다른 모든 처리들은 스택의 최상단에서 두 개 (혹은 처리에 따라 한 개 혹은 그 이상도 됩니다)의 요소들을 소비하며, 그 결과를 스택에 옮깁니다. 
물론, 스택에 좀 더 깊이 접근하기 위해 스택의 요소들을 스토리지로 옮길 수도 있으나, 스택의 첫 상단의 요소를 제거하지 않고 스택의 더 깊은 곳에 있는 임의의 요소에 접근하는 것은 불가능합니다.

.. index:: ! instruction

명령어 집합
===============

EVM의 명령어 집합은 합의 문제를 야기시킬 소지가 있는 부정확하거나 불규칙적인 시행을 최소화하기 위해 존재합니다. 
모든 명령어들은 기초 데이터 타입, 256비트의 단어 혹은 메모리의 일부분 (혹은 다른 바이트 배열들) 상에서 동작합니다. 
일반 기하학, 비트, 논리 연산자 및 비교 연산자도 있습니다. 조건부 혹은 비조건부 건너뛰기 또한 가능합니다. 
더불어, 컨트랙트들은 숫자, 타임 스탬프와 같이 현재 블록의 관련있는 프로프티들에게 접근 가능합니다. 

전체 리스트를 보시려면 inline assembly 문서 중 하나인  :ref:`list of opcodes <opcodes>` 를 참고하시기 바랍니다. 

.. index:: ! message call, function;call

Message Calls
=============

컨트랙트는 message call을 통하여 다른 컨트랙트를 호출하거나 컨트랙트와 연결되어 있지 않은 계정으로 Ether를 보낼 수 있습니다.
Message call은 소스, 타겟, 데이터 페이로드, Ether, 가스 그리고 데이터 반환이 있다는 점에서 트랜잭션과 비슷합니다.
사실상 모든 트랜잭션은 더 많은 mesage call들을 생성할 수 있는 상위 message call로 구성되어 있습니다.

컨트랙트는 내부 message call에게 남아있는 **가스**를 얼마만큼 보낼지, 그리고 얼마만큼 보존할지를 결정합니다.
만일 가스 부족 예외(혹은 기타 예외)가 내부 call에서 발생될 경우, 스택에 에러값으로 신호가 보내집니다.
이 경우, call과 함께 보내진 가스만이 사용됩니다. 
Solidity에선, 컨트랙트를 호출하는 것은 이러한 상황에서 기본적으로 수동적인 예외를 야기시켜서 콜 스택에 예외들이 많이 발생합니다.

이미 언급드렸다시피, 호출된 컨트랙트(호출자와 동일할 수도 있습니다)는 새롭게 정리된 메모리 인스턴스를 받게 되며
call payload에 접근할 수 있게 되는데, 이는 **calldata**라 하는 독립된 공간에서 제공됩니다.
해당 처리 건을 완료한 후, 호출자에 의해 미리 정해진 호출자의 메모리 장소에 저장될 데이터를 반환합니다.
이러한 모든 호출들은 전부 동기적입니다. 

콜들은 1024만큼의 depth로 **한정되어 있는데**, 이는 더욱 복잡한 작업의 경우 재귀 호출보다는 루프가 더 선호된다는 의미입니다.
또한, 63 혹은 64번째의 가스만 message call에 전달될 수 있으며 이는 실제론 1000보다 작은 depth limit이 걸리게 됩니다.

.. index:: delegatecall, callcode, library

Delegatecall / Callcode and Libraries
=====================================

**deleegatecall**이라 하는 message call의 특수 변형 형태가 있습니다. 
message call과 동일하지만 호출 중인 컨트랙트의 컨텍스트 내에서 실행되는 타겟 주소의 코드와 
``msg.sender`` 및 ``msg.value``가 그 값들을 변경하지 않는다는 점만 다릅니다. 

이는 컨트랙트가 런타임에서 다른 주소로부터 코드를 동적으로 로드할 수 있음을 의미합니다. 
스토리지, 현 주소 그리고 잔고는 여전히 호출 중인 컨트랙트를 참조하며 오직 코드만이 호출된 주소로부터 가져와집니다.

이렇게 함으로서 Solidity 내에 있는 "라이브러리" 특성(예를 들어 복잡한 데이터 구조를 시행하기 위한 컨트랙트 스토리지에 적용될 수 있는
재사용 가능한 라이브러리 코드)을 시행할 수 있게 만듭니다.

.. index:: log

Logs
====

It is possible to store data in a specially indexed data structure
that maps all the way up to the block level. This feature called **logs**
is used by Solidity in order to implement :ref:`events <events>`.
Contracts cannot access log data after it has been created, but they
can be efficiently accessed from outside the blockchain.
Since some part of the log data is stored in `bloom filters <https://en.wikipedia.org/wiki/Bloom_filter>`_, it is
possible to search for this data in an efficient and cryptographically
secure way, so network peers that do not download the whole blockchain
(so-called "light clients") can still find these logs.

.. index:: contract creation

Create
======

Contracts can even create other contracts using a special opcode (i.e.
they do not simply call the zero address as a transaction would). The only difference between
these **create calls** and normal message calls is that the payload data is
executed and the result stored as code and the caller / creator
receives the address of the new contract on the stack.

.. index:: selfdestruct, self-destruct, deactivate

Deactivate and Self-destruct
============================

The only way to remove code from the blockchain is when a contract at that
address performs the ``selfdestruct`` operation. The remaining Ether stored
at that address is sent to a designated target and then the storage and code
is removed from the state. Removing the contract in theory sounds like a good
idea, but it is potentially dangerous, as if someone sends Ether to removed
contracts, the Ether is forever lost.

.. warning::
    Even if a contract is removed by ``selfdestruct``, it is still part of the
    history of the blockchain and probably retained by most Ethereum nodes.
    So using ``selfdestruct`` is not the same as deleting data from a hard disk.

.. note::
    Even if a contract's code does not contain a call to ``selfdestruct``,
    it can still perform that operation using ``delegatecall`` or ``callcode``.

If you want to deactivate your contracts, you should instead **disable** them
by changing some internal state which causes all functions to revert. This
makes it impossible to use the contract, as it returns Ether immediately.


.. index:: ! precompiled contracts, ! precompiles, ! contract;precompiled

.. _precompiledContracts:

Precompiled Contracts
=====================

There is a small set of contract addresses that are special:
The address range between ``1`` and (including) ``8`` contains
"precompiled contracts" that can be called as any other contract
but their behaviour (and their gas consumption) is not defined
by EVM code stored at that address (they do not contain code)
but instead is implemented in the EVM execution environment itself.

Different EVM-compatible chains might use a different set of
precompiled contracts. It might also be possible that new
precompiled contracts are added to the Ethereum main chain in the future,
but you can reasonably expect them to always be in the range between
``1`` and ``0xffff`` (inclusive).
