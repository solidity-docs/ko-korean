.. _metadata:

#################
Contract Metadata
#################

.. index:: metadata, contract verification

The Solidity compiler automatically generates a JSON file, the contract
metadata, that contains information about the compiled contract. You can use
this file to query the compiler version, the sources used, the ABI and NatSpec
documentation to more safely interact with the contract and verify its source
code.

The compiler appends by default the IPFS hash of the metadata file to the end
of the bytecode (for details, see below) of each contract, so that you can
retrieve the file in an authenticated way without having to resort to a
centralized data provider. The other available options are the Swarm hash and
not appending the metadata hash to the bytecode.  These can be configured via
the :ref:`Standard JSON Interface<compiler-api>`.

You have to publish the metadata file to IPFS, Swarm, or another service so
that others can access it. You create the file by using the ``solc --metadata``
command together with the ``--output-dir`` parameter. Without the parameter,
the metadata will be written to standard output.
The metadata contains IPFS and Swarm references to the source code, so you have to
upload all source files in addition to the metadata file. For IPFS, the hash contained
in the CID returned by ``ipfs add`` (not the direct sha2-256 hash of the file)
shall match with the one contained in the bytecode.

The metadata file has the following format. The example below is presented in a
human-readable way. Properly formatted metadata should use quotes correctly,
reduce whitespace to a minimum and sort the keys of all objects to arrive at a
unique formatting. Comments are not permitted and used here only for
explanatory purposes.

.. code-block:: javascript

    {
      // Required: The version of the metadata format
      "version": "1",
      // Required: Source code language, basically selects a "sub-version"
      // of the specification
      "language": "Solidity",
      // Required: Details about the compiler, contents are specific
      // to the language.
      "compiler": {
        // Required for Solidity: Version of the compiler
        "version": "0.4.6+commit.2dabbdf0.Emscripten.clang",
        // Optional: Hash of the compiler binary which produced this output
        "keccak256": "0x123..."
      },
      // Required: Compilation source files/source units, keys are file names
      "sources":
      {
        "myFile.sol": {
          // Required: keccak256 hash of the source file
          "keccak256": "0x123...",
          // Required (unless "content" is used, see below): Sorted URL(s)
          // to the source file, protocol is more or less arbitrary, but a
          // Swarm URL is recommended
          "urls": [ "bzzr://56ab..." ],
          // Optional: SPDX license identifier as given in the source file
          "license": "MIT"
        },
        "destructible": {
          // Required: keccak256 hash of the source file
          "keccak256": "0x234...",
          // Required (unless "url" is used): literal contents of the source file
          "content": "contract destructible is owned { function destroy() { if (msg.sender == owner) selfdestruct(owner); } }"
        }
      },
      // Required: Compiler settings
      "settings":
      {
        // Required for Solidity: Sorted list of remappings
        "remappings": [ ":g=/dir" ],
        // Optional: Optimizer settings. The fields "enabled" and "runs" are deprecated
        // and are only given for backwards-compatibility.
        "optimizer": {
          "enabled": true,
          "runs": 500,
          "details": {
            // peephole defaults to "true"
            "peephole": true,
            // inliner defaults to "true"
            "inliner": true,
            // jumpdestRemover defaults to "true"
            "jumpdestRemover": true,
            "orderLiterals": false,
            "deduplicate": false,
            "cse": false,
            "constantOptimizer": false,
            "yul": true,
            // Optional: Only present if "yul" is "true"
            "yulDetails": {
              "stackAllocation": false,
              "optimizerSteps": "dhfoDgvulfnTUtnIf..."
            }
          }
        },
        "metadata": {
          // Reflects the setting used in the input json, defaults to false
          "useLiteralContent": true,
          // Reflects the setting used in the input json, defaults to "ipfs"
          "bytecodeHash": "ipfs"
        },
        // Required for Solidity: File and name of the contract or library this
        // metadata is created for.
        "compilationTarget": {
          "myFile.sol": "MyContract"
        },
        // Required for Solidity: Addresses for libraries used
        "libraries": {
          "MyLib": "0x123123..."
        }
      },
      // Required: Generated information about the contract.
      "output":
      {
        // Required: ABI definition of the contract
        "abi": [/* ... */],
        // Required: NatSpec user documentation of the contract
        "userdoc": [/* ... */],
        // Required: NatSpec developer documentation of the contract
        "devdoc": [/* ... */]
      }
    }

.. warning::
  Since the bytecode of the resulting contract contains the metadata hash by default, any
  change to the metadata might result in a change of the bytecode. This includes
  changes to a filename or path, and since the metadata includes a hash of all the
  sources used, a single whitespace change results in different metadata, and
  different bytecode.

.. note::
    The ABI definition above has no fixed order. It can change with compiler versions.
    Starting from Solidity version 0.5.12, though, the array maintains a certain
    order.

.. _encoding-of-the-metadata-hash-in-the-bytecode:

Encoding of the Metadata Hash in the Bytecode
=============================================

Because we might support other ways to retrieve the metadata file in the future,
the mapping ``{"ipfs": <IPFS hash>, "solc": <compiler version>}`` is stored
`CBOR <https://tools.ietf.org/html/rfc7049>`_-encoded. Since the mapping might
contain more keys (see below) and the beginning of that
encoding is not easy to find, its length is added in a two-byte big-endian
encoding. The current version of the Solidity compiler usually adds the following
to the end of the deployed bytecode

.. code-block:: text

    0xa2
    0x64 'i' 'p' 'f' 's' 0x58 0x22 <34 bytes IPFS hash>
    0x64 's' 'o' 'l' 'c' 0x43 <3 byte version encoding>
    0x00 0x33

So in order to retrieve the data, the end of the deployed bytecode can be checked
to match that pattern and use the IPFS hash to retrieve the file.

Whereas release builds of solc use a 3 byte encoding of the version as shown
above (one byte each for major, minor and patch version number), prerelease builds
will instead use a complete version string including commit hash and build date.

.. note::
  The CBOR mapping can also contain other keys, so it is better to fully
  decode the data instead of relying on it starting with ``0xa264``.
  For example, if any experimental features that affect code generation
  are used, the mapping will also contain ``"experimental": true``.

.. note::
  The compiler currently uses the IPFS hash of the metadata by default, but
  it may also use the bzzr1 hash or some other hash in the future, so do
  not rely on this sequence to start with ``0xa2 0x64 'i' 'p' 'f' 's'``.  We
  might also add additional data to this CBOR structure, so the best option
  is to use a proper CBOR parser.


Usage for Automatic Interface Generation and NatSpec
====================================================

The metadata is used in the following way: A component that wants to interact
with a contract (e.g. Mist or any wallet) retrieves the code of the contract,
from that the IPFS/Swarm hash of a file which is then retrieved.  That file
is JSON-decoded into a structure like above.

The component can then use the ABI to automatically generate a rudimentary
user interface for the contract.

Furthermore, the wallet can use the NatSpec user documentation to display a confirmation message to the user
whenever they interact with the contract, together with requesting
authorization for the transaction signature.

For additional information, read :doc:`Ethereum Natural Language Specification (NatSpec) format <natspec-format>`.

Usage for Source Code Verification
==================================

In order to verify the compilation, sources can be retrieved from IPFS/Swarm
via the link in the metadata file.
The compiler of the correct version (which is checked to be part of the "official" compilers)
is invoked on that input with the specified settings. The resulting
bytecode is compared to the data of the creation transaction or ``CREATE`` opcode data.
This automatically verifies the metadata since its hash is part of the bytecode.
Excess data corresponds to the constructor input data, which should be decoded
according to the interface and presented to the user.

In the repository `sourcify <https://github.com/ethereum/sourcify>`_
(`npm package <https://www.npmjs.com/package/source-verify>`_) you can see
example code that shows how to use this feature.
