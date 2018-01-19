# DocChain

Like Google Docs, this program provides a blockchain-based document-sharing App.

1. Implemented by Qt, OpenSSL and asynchronous libevent on Xcode.

2. Decentralized distributed database with incorruptible chain.

3. Real-time reaction for network nodes' document modification.

4. Cross-platform framework with iOS/Android Apps and desktop application.

5. Includes following features: "Publish Public Key", "Publish Address", "Sign Network Protocol", "Verify Remote Block", and "Encode Document Modification."

# Overview
![alt text](https://user-images.githubusercontent.com/13886288/35134627-7d159160-fd12-11e7-961b-3b8bda29aab4.png)

1. Local address: Identity of this device

2. Remote addresses: Show all connected remote nodes' addresses

3. Real-time document-sharing editor: User can modify document here, with remote nodes' modification updated here simultaneously

4. Remote modification: Character in red indicates the latest modification coming from remote node

5. Debugging status: Show some network protocols and useful messages

# Functionality

[Publish Public Key]

    Once the program is launched, public/private key pairs will be generated by ECC (Elliptic-curve cryptography) secp521r1 algorithm.

    Then, REMOTE_COMMAND_REPLY_PUBKEY and GET_PUBKEY will be broadcast by following format:

        REMOTE_COMMAND_REPLY_PUBKEY NodeAddress PublicKeyPlainFormat
        GET_PUBKEY

    When other nodes received neighbor's public key, this public key will be stored in a hash table with (key, value) = (NodeAddress, PublicKey).

    When other nodes received "GET_PUBKEY", its public key will be sent back by:
    
        REMOTE_COMMAND_REPLY_PUBKEY NodeAddress PublicKeyPlainFormat

[Publish Address]

    After public key is generated, address is then generated by RIPEMD160(SHA256(PublicKeyPlainFormat)).

    The address will be used inside network protocol.

[Sign Network Protocol]

    Here is network protocol:

        [BLOCKCHAIN_HEADER][ADDRESS][MessageLen][Message][Signature]
            BLOCKCHAIN_HEADER(strlen(BLOCKCHAIN_HEADER)): "__ThisIsBlockChainPacketByClarkYang__", to avoid conflicting to other network packets
            Address(40): Identify the user of this protocol
            MessageLen(4): (0000 ~ FFFF) Indicate message length
            Message(MessageLen): Transferred message
            Signature(-) : Message signature

    On sender node, message signature will be generated by the private key before sending.

    On receiver node, signature will be verified by the public key of sender (should already stored in hash table).
    
    Only the verified message will be executed; otherwise it'll be dropped.

[Verify Remote Block]

    When user modifies the document, a new block (encoded document modification) will be added to the tail of its blockchain.

    Then, this new block will be broadcast to request other nodes to add this new block (modification).

    When a node received a remote block, it will verify if this block is valid with current blockchain.

    Following criterias must be met before adding remote block to current blockchain:

        1. Remote block's index = current blockchain's latest index + 1
        2. Remote block's previous hash = current blockchain's latest hash
        3. Remote block's hash = SHA256(blockIndex, previousHash, timeStamp, modifyCode)

    If everything goes well, this remote block will be added to the tail of current blockchain.

[Encode Document Modification]

    Here is the encoded format of document modification:

        [MODE][POS][MODIFICATION]
            MODE(1): '+' (Add a character), '-' (Remove a character)
            POS(4): (0000 ~ FFFF) Indicate modification position
            MODIFICATION(-): Modification word(s)

    Normally, when user add a character from GUI, the '+' command will be broadcast.

    For example, "+003Fb" will be broadcast to indicate character 'b' is inserted to position 3F(63 in decimal).

    Similar, when user remove a character from GUI, the '-' command (ex. -003Fx) will be broadcast.

    Besides, if verifying remote block failed, we might ask for whole remote blockchain and replace the current one.

    In this situation, every remote blocks will be executed one-by-one, to reconstruct the same document as remote node.
