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

Overview
========

The Ethereum Virtual Machine or EVM is the runtime environment
for smart contracts in Ethereum. It is not only sandboxed but
actually completely isolated, which means that code running
inside the EVM has no access to network, filesystem or other processes.
Smart contracts even have limited access to other smart contracts.

.. index:: ! account, address, storage, balance

.. _accounts:

Accounts
========

There are two kinds of accounts in Ethereum which share the same
address space: **External accounts** that are controlled by
public-private key pairs (i.e. humans) and **contract accounts** which are
controlled by the code stored together with the account.

The address of an external account is determined from
the public key while the address of a contract is
determined at the time the contract is created
(it is derived from the creator address and the number
of transactions sent from that address, the so-called "nonce").

Regardless of whether or not the account stores code, the two types are
treated equally by the EVM.

Every account has a persistent key-value store mapping 256-bit words to 256-bit
words called **storage**.

Furthermore, every account has a **balance** in
Ether (in "Wei" to be exact, ``1 ether`` is ``10**18 wei``) which can be modified by sending transactions that
include Ether.

.. index:: ! transaction

Transactions
============

A transaction is a message that is sent from one account to another
account (which might be the same or empty, see below).
It can include binary data (which is called "payload") and Ether.

If the target account contains code, that code is executed and
the payload is provided as input data.

If the target account is not set (the transaction does not have
a recipient or the recipient is set to ``null``), the transaction
creates a **new contract**.
As already mentioned, the address of that contract is not
the zero address but an address derived from the sender and
its number of transactions sent (the "nonce"). The payload
of such a contract creation transaction is taken to be
EVM bytecode and executed. The output data of this execution is
permanently stored as the code of the contract.
This means that in order to create a contract, you do not
send the actual code of the contract, but in fact code that
returns that code when executed.

.. note::
  While a contract is being created, its code is still empty.
  Because of that, you should not call back into the
  contract under construction until its constructor has
  finished executing.

.. index:: ! gas, ! gas price

Gas
===

Upon creation, each transaction is charged with a certain amount of **gas**,
whose purpose is to limit the amount of work that is needed to execute
the transaction and to pay for this execution at the same time. While the EVM executes the
transaction, the gas is gradually depleted according to specific rules.

The **gas price** is a value set by the creator of the transaction, who
has to pay ``gas_price * gas`` up front from the sending account.
If some gas is left after the execution, it is refunded to the creator in the same way.

If the gas is used up at any point (i.e. it would be negative),
an out-of-gas exception is triggered, which reverts all modifications
made to the state in the current call frame.

.. index:: ! storage, ! memory, ! stack

Storage, Memory and the Stack
=============================

The Ethereum Virtual Machine has three areas where it can store data-
storage, memory and the stack, which are explained in the following
paragraphs.

Each account has a data area called **storage**, which is persistent between function calls
and transactions.
Storage is a key-value store that maps 256-bit words to 256-bit words.
It is not possible to enumerate storage from within a contract, it is
comparatively costly to read, and even more to initialise and modify storage. Because of this cost,
you should minimize what you store in persistent storage to what the contract needs to run.
Store data like derived calculations, caching, and aggregates outside of the contract.
A contract can neither read nor write to any storage apart from its own.

The second data area is called **memory**, of which a contract obtains
a freshly cleared instance for each message call. Memory is linear and can be
addressed at byte level, but reads are limited to a width of 256 bits, while writes
can be either 8 bits or 256 bits wide. Memory is expanded by a word (256-bit), when
accessing (either reading or writing) a previously untouched memory word (i.e. any offset
within a word). At the time of expansion, the cost in gas must be paid. Memory is more
costly the larger it grows (it scales quadratically).

The EVM is not a register machine but a stack machine, so all
computations are performed on a data area called the **stack**. It has a maximum size of
1024 elements and contains words of 256 bits. Access to the stack is
limited to the top end in the following way:
It is possible to copy one of
the topmost 16 elements to the top of the stack or swap the
topmost element with one of the 16 elements below it.
All other operations take the topmost two (or one, or more, depending on
the operation) elements from the stack and push the result onto the stack.
Of course it is possible to move stack elements to storage or memory
in order to get deeper access to the stack,
but it is not possible to just access arbitrary elements deeper in the stack
without first removing the top of the stack.

.. index:: ! instruction

Instruction Set
===============

The instruction set of the EVM is kept minimal in order to avoid
incorrect or inconsistent implementations which could cause consensus problems.
All instructions operate on the basic data type, 256-bit words or on slices of memory
(or other byte arrays).
The usual arithmetic, bit, logical and comparison operations are present.
Conditional and unconditional jumps are possible. Furthermore,
contracts can access relevant properties of the current block
like its number and timestamp.

For a complete list, please see the :ref:`list of opcodes <opcodes>` as part of the inline
assembly documentation.

.. index:: ! message call, function;call

Message Calls
=============

Contracts can call other contracts or send Ether to non-contract
accounts by the means of message calls. Message calls are similar
to transactions, in that they have a source, a target, data payload,
Ether, gas and return data. In fact, every transaction consists of
a top-level message call which in turn can create further message calls.

A contract can decide how much of its remaining **gas** should be sent
with the inner message call and how much it wants to retain.
If an out-of-gas exception happens in the inner call (or any
other exception), this will be signaled by an error value put onto the stack.
In this case, only the gas sent together with the call is used up.
In Solidity, the calling contract causes a manual exception by default in
such situations, so that exceptions "bubble up" the call stack.

As already said, the called contract (which can be the same as the caller)
will receive a freshly cleared instance of memory and has access to the
call payload - which will be provided in a separate area called the **calldata**.
After it has finished execution, it can return data which will be stored at
a location in the caller's memory preallocated by the caller.
All such calls are fully synchronous.

Calls are **limited** to a depth of 1024, which means that for more complex
operations, loops should be preferred over recursive calls. Furthermore,
only 63/64th of the gas can be forwarded in a message call, which causes a
depth limit of a little less than 1000 in practice.

.. index:: delegatecall, callcode, library

Delegatecall / Callcode and Libraries
=====================================

There exists a special variant of a message call, named **delegatecall**
which is identical to a message call apart from the fact that
the code at the target address is executed in the context of the calling
contract and ``msg.sender`` and ``msg.value`` do not change their values.

This means that a contract can dynamically load code from a different
address at runtime. Storage, current address and balance still
refer to the calling contract, only the code is taken from the called address.

This makes it possible to implement the "library" feature in Solidity:
Reusable library code that can be applied to a contract's storage, e.g. in
order to implement a complex data structure.

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
