.. index:: ! contract;creation, constructor

******************
Creating Contracts
******************

컨트랙트는 이더리움 트랜잭션를 통한 "외부로부터" 또는 솔리디티의 컨트랙트 내부로부터
생성될 수 있습니다.

`Remix <https://remix.ethereum.org/>`_와 같은 IDE들은 UI 요소들을 활용하여
생성 프로세스를 원활하게 만듭니다.

이더리움에서 프로그래밍 방식으로 컨트랙트를 생성할 수 있는 한 가지 방법은 JavaScript API인
`web3.js <https://github.com/ethereum/web3.js>`_를 통한 방법입니다.
이는 컨트랙트 생성을 용이하게 하기 위해 `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_ 라고 불리는 함수를 갖고 있습니다.
    
컨트랙트가 생성되었을 때, :ref:`constructor <constructor>` (``constructor`` 키워드와 함께 정의된 함수)
가 한 번 호출됩니다.

생성자는 선택적입니다. 오직 한 개의 생성자만 허용되는데, 이는 오버로딩이 제공되지 않음을 의미합니다.

생성자가 실행된 후, 컨트랙트의 최종 코드는 블록체인 위에 저장됩니다. 이 코드는 모든
퍼블릭, 그리고 외부 함수들과 함수 호출을 통해 도달할 수 있는 모든 함수들을 포함하고 있습니다.
배포된 코드는 생성자 코드나 혹은 생성자로부터만 호출되는 내부 함수들을 포함하지 않습니다.

.. index:: constructor;arguments

내부적으로, 생성자 인수는 컨트랙트 자체의 코드 뒤에 :ref:`ABI encoded <ABI>`로 전달되지만
``web3.js``를 사용하는 경우 이를 신경 쓸 필요가 없습니다.

만약 컨트랙트가 다른 컨트랙트를 만들기를 원한다면, 생성된 컨트랙트의 소스 코드 (그리고 바이너리)를
작성자에게 알려야 합니다. 이는 순환 생성 종속성이 불가능함을 의미합니다.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;


    contract OwnedToken {
        // `TokenCreator` is a contract type that is defined below.
        // It is fine to reference it as long as it is not used
        // to create a new contract.
        TokenCreator creator;
        address owner;
        bytes32 name;

        // This is the constructor which registers the
        // creator and the assigned name.
        constructor(bytes32 _name) {
            // State variables are accessed via their name
            // and not via e.g. `this.owner`. Functions can
            // be accessed directly or through `this.f`,
            // but the latter provides an external view
            // to the function. Especially in the constructor,
            // you should not access functions externally,
            // because the function does not exist yet.
            // See the next section for details.
            owner = msg.sender;

            // We perform an explicit type conversion from `address`
            // to `TokenCreator` and assume that the type of
            // the calling contract is `TokenCreator`, there is
            // no real way to verify that.
            // This does not create a new contract.
            creator = TokenCreator(msg.sender);
            name = _name;
        }

        function changeName(bytes32 newName) public {
            // Only the creator can alter the name.
            // We compare the contract based on its
            // address which can be retrieved by
            // explicit conversion to address.
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            // Only the current owner can transfer the token.
            if (msg.sender != owner) return;

            // We ask the creator contract if the transfer
            // should proceed by using a function of the
            // `TokenCreator` contract defined below. If
            // the call fails (e.g. due to out-of-gas),
            // the execution also fails here.
            if (creator.isTokenTransferOK(owner, newOwner))
                owner = newOwner;
        }
    }


    contract TokenCreator {
        function createToken(bytes32 name)
            public
            returns (OwnedToken tokenAddress)
        {
            // Create a new `Token` contract and return its address.
            // From the JavaScript side, the return type
            // of this function is `address`, as this is
            // the closest type available in the ABI.
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name) public {
            // Again, the external type of `tokenAddress` is
            // simply `address`.
            tokenAddress.changeName(name);
        }

        // Perform checks to determine if transferring a token to the
        // `OwnedToken` contract should proceed
        function isTokenTransferOK(address currentOwner, address newOwner)
            public
            pure
            returns (bool ok)
        {
            // Check an arbitrary condition to see if transfer should proceed
            return keccak256(abi.encodePacked(currentOwner, newOwner))[0] == 0x7f;
        }
    }
