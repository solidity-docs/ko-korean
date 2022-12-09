.. index:: Bugs

.. _known_bugs:

##################
알려진 버그들
##################

아래는 솔리디티 컴파일러에서 발견된 보안과 관련된 버그들을 JSON 형태의 리스트로 나타낸 것입니다. 
파일은 `Github 레포지토리<https://github.com/ethereum/solidity/blob/develop/docs/bugs.json>`_ 에서 호스팅되고 있습니다.
리스트는 과거 버전 0.3.0서부터 나타내고 있으며 버그는 리스트에 나타나있지 않은 전 버전에서만 존재합니다.

`bugs_by_version.json
<https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json>`_라는 또다른 파일이 있으며,
이는 컴파일러의 어떤 버전에 영향을 끼치는지 체크하는데 사용될 수 있습니다.

컨트랙트와 상호작용하고 있는 컨트랙트 소스 유효성 검사 툴과 그밖의 툴들은 반드시 다음 조건들을 따라 리스트에 표기해야 합니다.

- It is mildly suspicious if a contract was compiled with a nightly
  compiler version instead of a released version. This list does not keep
  track of unreleased or nightly versions.
- It is also mildly suspicious if a contract was compiled with a version that was
  not the most recent at the time the contract was created. For contracts
  created from other contracts, you have to follow the creation chain
  back to a transaction and use the date of that transaction as creation date.
- It is highly suspicious if a contract was compiled with a compiler that
  contains a known bug and the contract was created at a time where a newer
  compiler version containing a fix was already released.

The JSON file of known bugs below is an array of objects, one for each bug,
with the following keys:

uid
    Unique identifier given to the bug in the form of ``SOL-<year>-<number>``.
    It is possible that multiple entries exists with the same uid. This means
    multiple version ranges are affected by the same bug.
name
    Unique name given to the bug
summary
    Short description of the bug
description
    Detailed description of the bug
link
    URL of a website with more detailed information, optional
introduced
    The first published compiler version that contained the bug, optional
fixed
    The first published compiler version that did not contain the bug anymore
publish
    The date at which the bug became known publicly, optional
severity
    Severity of the bug: very low, low, medium, high. Takes into account
    discoverability in contract tests, likelihood of occurrence and
    potential damage by exploits.
conditions
    Conditions that have to be met to trigger the bug. The following
    keys can be used:
    ``optimizer``, Boolean value which
    means that the optimizer has to be switched on to enable the bug.
    ``evmVersion``, a string that indicates which EVM version compiler
    settings trigger the bug. The string can contain comparison
    operators. For example, ``">=constantinople"`` means that the bug
    is present when the EVM version is set to ``constantinople`` or
    later.
    If no conditions are given, assume that the bug is present.
check
    This field contains different checks that report whether the smart contract
    contains the bug or not. The first type of check are Javascript regular
    expressions that are to be matched against the source code ("source-regex")
    if the bug is present.  If there is no match, then the bug is very likely
    not present. If there is a match, the bug might be present.  For improved
    accuracy, the checks should be applied to the source code after stripping
    comments.
    The second type of check are patterns to be checked on the compact AST of
    the Solidity program ("ast-compact-json-path"). The specified search query
    is a `JsonPath <https://github.com/json-path/JsonPath>`_ expression.
    If at least one path of the Solidity AST matches the query, the bug is
    likely present.

.. literalinclude:: bugs.json
   :language: js
