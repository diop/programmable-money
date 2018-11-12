# The Ethereum Blockchain Curriculum
by Fodé Diop

# Motivation
```
This project is heavily influenced by PyClass from Noisebridge. 
The PM workshop will run every Monday for 6 consecutive weeks, except for when it falls on a holiday.
```

* First Monday: **Introduction to Solidity and Smart Contracts - Deploy your own Coin**
* Second Monday: **Ethereum Blockchain Transactions Fundamentals**
* Third Monday: **Web3JS aka enough Javascript to be dangerous**
* Fourth Monday: **Writing Secure Smart Contracts**
* Fifth Monday: **Dapp Frontends, Deployments, and a dash more Javascript**
* Sixth Monday: **What to build?**
* Monday Break: **Special Guests**

### Solidity and Smart Contracts
[Ethereum EIPs](https://github.com/ethereum/EIPs)
```
**EIP** stands for **Ethereum Improvement Proposal**. An EIP is a design document providing information to the Ethereum community, or describing a new feature for Ethereum or its processes or environment.
```
* [ERC-20 EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)
* [ERC-721 EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md)

```js
// ----------------------------------------------------------------------------
// ERC Token Standard #20 Interface
// https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
// ----------------------------------------------------------------------------
contract ERC20Interface {
    function totalSupply() public constant returns (uint);
    function balanceOf(address tokenOwner) public constant returns (uint balance);
    function allowance(address tokenOwner, address spender) public constant returns (uint remaining);
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```

```js
pragma solidity ^0.4.24;

// ----------------------------------------------------------------------------
// 'FIXED' 'Example Fixed Supply Token' token contract
//
// Symbol      : FIXED
// Name        : Example Fixed Supply Token
// Total supply: 1,000,000.000000000000000000
// Decimals    : 18
//
// Enjoy.
//
// (c) BokkyPooBah / Bok Consulting Pty Ltd 2018. The MIT Licence.
// ----------------------------------------------------------------------------


// ----------------------------------------------------------------------------
// Safe maths
// ----------------------------------------------------------------------------
library SafeMath {
    function add(uint a, uint b) internal pure returns (uint c) {
        c = a + b;
        require(c >= a);
    }
    function sub(uint a, uint b) internal pure returns (uint c) {
        require(b <= a);
        c = a - b;
    }
    function mul(uint a, uint b) internal pure returns (uint c) {
        c = a * b;
        require(a == 0 || c / a == b);
    }
    function div(uint a, uint b) internal pure returns (uint c) {
        require(b > 0);
        c = a / b;
    }
}


// ----------------------------------------------------------------------------
// ERC Token Standard #20 Interface
// https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
// ----------------------------------------------------------------------------
contract ERC20Interface {
    function totalSupply() public constant returns (uint);
    function balanceOf(address tokenOwner) public constant returns (uint balance);
    function allowance(address tokenOwner, address spender) public constant returns (uint remaining);
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}


// ----------------------------------------------------------------------------
// Contract function to receive approval and execute function in one call
//
// Borrowed from MiniMeToken
// ----------------------------------------------------------------------------
contract ApproveAndCallFallBack {
    function receiveApproval(address from, uint256 tokens, address token, bytes data) public;
}


// ----------------------------------------------------------------------------
// Owned contract
// ----------------------------------------------------------------------------
contract Owned {
    address public owner;
    address public newOwner;

    event OwnershipTransferred(address indexed _from, address indexed _to);

    constructor() public {
        owner = msg.sender;
    }

    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }

    function transferOwnership(address _newOwner) public onlyOwner {
        newOwner = _newOwner;
    }
    function acceptOwnership() public {
        require(msg.sender == newOwner);
        emit OwnershipTransferred(owner, newOwner);
        owner = newOwner;
        newOwner = address(0);
    }
}


// ----------------------------------------------------------------------------
// ERC20 Token, with the addition of symbol, name and decimals and a
// fixed supply
// ----------------------------------------------------------------------------
contract FixedSupplyToken is ERC20Interface, Owned {
    using SafeMath for uint;

    string public symbol;
    string public  name;
    uint8 public decimals;
    uint _totalSupply;

    mapping(address => uint) balances;
    mapping(address => mapping(address => uint)) allowed;


    // ------------------------------------------------------------------------
    // Constructor
    // ------------------------------------------------------------------------
    constructor() public {
        symbol = "FIXED";
        name = "Example Fixed Supply Token";
        decimals = 18;
        _totalSupply = 1000000 * 10**uint(decimals);
        balances[owner] = _totalSupply;
        emit Transfer(address(0), owner, _totalSupply);
    }


    // ------------------------------------------------------------------------
    // Total supply
    // ------------------------------------------------------------------------
    function totalSupply() public view returns (uint) {
        return _totalSupply.sub(balances[address(0)]);
    }


    // ------------------------------------------------------------------------
    // Get the token balance for account `tokenOwner`
    // ------------------------------------------------------------------------
    function balanceOf(address tokenOwner) public view returns (uint balance) {
        return balances[tokenOwner];
    }


    // ------------------------------------------------------------------------
    // Transfer the balance from token owner's account to `to` account
    // - Owner's account must have sufficient balance to transfer
    // - 0 value transfers are allowed
    // ------------------------------------------------------------------------
    function transfer(address to, uint tokens) public returns (bool success) {
        balances[msg.sender] = balances[msg.sender].sub(tokens);
        balances[to] = balances[to].add(tokens);
        emit Transfer(msg.sender, to, tokens);
        return true;
    }


    // ------------------------------------------------------------------------
    // Token owner can approve for `spender` to transferFrom(...) `tokens`
    // from the token owner's account
    //
    // https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md
    // recommends that there are no checks for the approval double-spend attack
    // as this should be implemented in user interfaces 
    // ------------------------------------------------------------------------
    function approve(address spender, uint tokens) public returns (bool success) {
        allowed[msg.sender][spender] = tokens;
        emit Approval(msg.sender, spender, tokens);
        return true;
    }


    // ------------------------------------------------------------------------
    // Transfer `tokens` from the `from` account to the `to` account
    // 
    // The calling account must already have sufficient tokens approve(...)-d
    // for spending from the `from` account and
    // - From account must have sufficient balance to transfer
    // - Spender must have sufficient allowance to transfer
    // - 0 value transfers are allowed
    // ------------------------------------------------------------------------
    function transferFrom(address from, address to, uint tokens) public returns (bool success) {
        balances[from] = balances[from].sub(tokens);
        allowed[from][msg.sender] = allowed[from][msg.sender].sub(tokens);
        balances[to] = balances[to].add(tokens);
        emit Transfer(from, to, tokens);
        return true;
    }


    // ------------------------------------------------------------------------
    // Returns the amount of tokens approved by the owner that can be
    // transferred to the spender's account
    // ------------------------------------------------------------------------
    function allowance(address tokenOwner, address spender) public view returns (uint remaining) {
        return allowed[tokenOwner][spender];
    }


    // ------------------------------------------------------------------------
    // Token owner can approve for `spender` to transferFrom(...) `tokens`
    // from the token owner's account. The `spender` contract function
    // `receiveApproval(...)` is then executed
    // ------------------------------------------------------------------------
    function approveAndCall(address spender, uint tokens, bytes data) public returns (bool success) {
        allowed[msg.sender][spender] = tokens;
        emit Approval(msg.sender, spender, tokens);
        ApproveAndCallFallBack(spender).receiveApproval(msg.sender, tokens, this, data);
        return true;
    }


    // ------------------------------------------------------------------------
    // Don't accept ETH
    // ------------------------------------------------------------------------
    function () public payable {
        revert();
    }


    // ------------------------------------------------------------------------
    // Owner can transfer out any accidentally sent ERC20 tokens
    // ------------------------------------------------------------------------
    function transferAnyERC20Token(address tokenAddress, uint tokens) public onlyOwner returns (bool success) {
        return ERC20Interface(tokenAddress).transfer(owner, tokens);
    }
}
```

```js
 pragma solidity ^0.4.20;

/// @title ERC-721 Non-Fungible Token Standard
/// @dev See https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md
///  Note: the ERC-165 identifier for this interface is 0x80ac58cd
interface ERC721 /* is ERC165 */ {
    /// @dev This emits when ownership of any NFT changes by any mechanism.
    ///  This event emits when NFTs are created (`from` == 0) and destroyed
    ///  (`to` == 0). Exception: during contract creation, any number of NFTs
    ///  may be created and assigned without emitting Transfer. At the time of
    ///  any transfer, the approved address for that NFT (if any) is reset to none.
    event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);

    /// @dev This emits when the approved address for an NFT is changed or
    ///  reaffirmed. The zero address indicates there is no approved address.
    ///  When a Transfer event emits, this also indicates that the approved
    ///  address for that NFT (if any) is reset to none.
    event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

    /// @dev This emits when an operator is enabled or disabled for an owner.
    ///  The operator can manage all NFTs of the owner.
    event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);

    /// @notice Count all NFTs assigned to an owner
    /// @dev NFTs assigned to the zero address are considered invalid, and this
    ///  function throws for queries about the zero address.
    /// @param _owner An address for whom to query the balance
    /// @return The number of NFTs owned by `_owner`, possibly zero
    function balanceOf(address _owner) external view returns (uint256);

    /// @notice Find the owner of an NFT
    /// @dev NFTs assigned to zero address are considered invalid, and queries
    ///  about them do throw.
    /// @param _tokenId The identifier for an NFT
    /// @return The address of the owner of the NFT
    function ownerOf(uint256 _tokenId) external view returns (address);

    /// @notice Transfers the ownership of an NFT from one address to another address
    /// @dev Throws unless `msg.sender` is the current owner, an authorized
    ///  operator, or the approved address for this NFT. Throws if `_from` is
    ///  not the current owner. Throws if `_to` is the zero address. Throws if
    ///  `_tokenId` is not a valid NFT. When transfer is complete, this function
    ///  checks if `_to` is a smart contract (code size > 0). If so, it calls
    ///  `onERC721Received` on `_to` and throws if the return value is not
    ///  `bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"))`.
    /// @param _from The current owner of the NFT
    /// @param _to The new owner
    /// @param _tokenId The NFT to transfer
    /// @param data Additional data with no specified format, sent in call to `_to`
    function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data) external payable;

    /// @notice Transfers the ownership of an NFT from one address to another address
    /// @dev This works identically to the other function with an extra data parameter,
    ///  except this function just sets data to ""
    /// @param _from The current owner of the NFT
    /// @param _to The new owner
    /// @param _tokenId The NFT to transfer
    function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable;

    /// @notice Transfer ownership of an NFT -- THE CALLER IS RESPONSIBLE
    ///  TO CONFIRM THAT `_to` IS CAPABLE OF RECEIVING NFTS OR ELSE
    ///  THEY MAY BE PERMANENTLY LOST
    /// @dev Throws unless `msg.sender` is the current owner, an authorized
    ///  operator, or the approved address for this NFT. Throws if `_from` is
    ///  not the current owner. Throws if `_to` is the zero address. Throws if
    ///  `_tokenId` is not a valid NFT.
    /// @param _from The current owner of the NFT
    /// @param _to The new owner
    /// @param _tokenId The NFT to transfer
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;

    /// @notice Set or reaffirm the approved address for an NFT
    /// @dev The zero address indicates there is no approved address.
    /// @dev Throws unless `msg.sender` is the current NFT owner, or an authorized
    ///  operator of the current owner.
    /// @param _approved The new approved NFT controller
    /// @param _tokenId The NFT to approve
    function approve(address _approved, uint256 _tokenId) external payable;

    /// @notice Enable or disable approval for a third party ("operator") to manage
    ///  all of `msg.sender`'s assets.
    /// @dev Emits the ApprovalForAll event. The contract MUST allow
    ///  multiple operators per owner.
    /// @param _operator Address to add to the set of authorized operators.
    /// @param _approved True if the operator is approved, false to revoke approval
    function setApprovalForAll(address _operator, bool _approved) external;

    /// @notice Get the approved address for a single NFT
    /// @dev Throws if `_tokenId` is not a valid NFT
    /// @param _tokenId The NFT to find the approved address for
    /// @return The approved address for this NFT, or the zero address if there is none
    function getApproved(uint256 _tokenId) external view returns (address);

    /// @notice Query if an address is an authorized operator for another address
    /// @param _owner The address that owns the NFTs
    /// @param _operator The address that acts on behalf of the owner
    /// @return True if `_operator` is an approved operator for `_owner`, false otherwise
    function isApprovedForAll(address _owner, address _operator) external view returns (bool);
}

interface ERC165 {
    /// @notice Query if a contract implements an interface
    /// @param interfaceID The interface identifier, as specified in ERC-165
    /// @dev Interface identification is specified in ERC-165. This function
    ///  uses less than 30,000 gas.
    /// @return `true` if the contract implements `interfaceID` and
    ///  `interfaceID` is not 0xffffffff, `false` otherwise
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}

interface ERC721TokenReceiver {
    /// @notice Handle the receipt of an NFT
    /// @dev The ERC721 smart contract calls this function on the
    /// recipient after a `transfer`. This function MAY throw to revert and reject the transfer. Return
    /// of other than the magic value MUST result in the transaction being reverted.
    /// @notice The contract address is always the message sender.
    /// @param _operator The address which called `safeTransferFrom` function
    /// @param _from The address which previously owned the token
    /// @param _tokenId The NFT identifier which is being transferred
    /// @param _data Additional data with no specified format
    /// @return `bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"))`
    /// unless throwing
    function onERC721Received(address _operator, address _from, uint256 _tokenId, bytes _data) external returns(bytes4);
}       
```

### Ethereum Blockchain Transactions
+ [Remix IDE](https://remix.ethereum.org/)
+ [Infura Blockchain as a Service](https://infura.io/)
+ [Etherscan Accounts](https://etherscan.io/accounts)

+ It's absolutely crucial to know how transactions work in Ethereum.

* Build a transaction

```js
web3.eth.getTransactionCount(acct1, (err, txCount) => {
    const txObject = {
        nonce: web3.utils.toHex(txCount),
        to: acct2,
        value: web3.utis.toHex(web3.utils.toWei('0.1', 'ether')),
        gasLimit: web3.utils.toHex(21000),
        gasPrice:  web3.utils.toHex(web3.utils.toWei('10', 'gwei'))
    }

    web3.eth.sendSignedtransaction(raw, (err, txHash) => {
        console.log('Transaction Hash: ', txHash)
    }) 
})
```
* Using Ganache

```js
$ npm install ethereumjs-tx
$ node
> const Web3 = require('web3')
> const web3 = new Web3('http://127.0.0.1:7545')
> const acct1 = '0x627306090abaB3A6e1400e9345bC60c78a8BEf57'
> const acct2 = '0xf17f52151EbEF6C7334FAD080c5704D77216b732'
> web3.eth.sendTransaction({ from: acct1, to: acct, value: web3.utils.toWei('1', 'ether') })
> web3.eth.getBalance(acct1)
> web3.eth.personal.unlockAccount --> On Geth
```

+ When you send transaction on Ganache, those accounts are unlocked. You need to sign your transaction first before you can
broadcast them.

+ Keep the private keys in envirnment variables for security.

```js
$ node
> const privateKey1 = Buffer.from(process.env.PRIVATE_KEY_1)
> const privateKey2 = Buffer.from(process.env.PRIVATE_KEY_2)
```

### Writing Secure Smart Contracts
+ [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-solidity)
+ [Mocha](https://mochajs.org/)
+ [Chai](https://www.chaijs.com/)

### Web3.js aka enough Javascript to be dangerous.
+ [Web3 1.0 ES6](https://github.com/ethereum/web3.js/tree/1.0ES6)
+ [Web3 Docs](https://web3js.readthedocs.io/en/1.0/index.html)

``` $ node -v ``` to check NodeJS version

+ To play with Web3 via NodeJS:

```js
$ node
> const Web3 = require('web3')
> const url = 'https://mainnet.infura.io/CTNrMRz6lyyxOxddWG7y'
> const web3 = new Web3(url)
> const address = '0x742d35Cc6634C0532925a3b844Bc454e4438f44e'
> web3.eth.getBalance(address, (err, bal) => { balance = bal})
> web3.utils.fromWei(balance, 'ether')
> balance
> web3.eth.accounts.create()
```

+ To get the balance from your local Ganache instance:

```js
$ node 
> web3.eth.getBalance('GANACHE_ACCOUNT_ADDRESS', (err, wei) => {balance = web3.utils.fromWei(wei, 'ether')})
> balance 
```

+ ``` web3.eth.Contract ``` creates an abstraction of our Smart Contract we can abstract with in Javascript.
+ to interact with a contract on the main Blockchain. Same steps to connect to infura plus:

```js
$ node 
> const abi = ABI_FROM_ETHERSCAN
> abi
> const contractAddress = CONTRACT_ADDRESS
> const contract = new web3.eth.Contract(abi, contractAddress)
> contract
> contract.methods
> contracts.methods.name().call((err, result) => { console.log(result) })
> contracts.methods.symbol().call((err, result) => { console.log(result) })
> contract.methods.totalSupply().call((err, result) => { console.log(result) })
> const accountAddress = ACCOUNT_ADDRESS_FROM_HOLDERS_LIST
```

### Dapp Frontends
[Truffle Frameworks](https://truffleframework.com/truffle)

### What to build?
[0x Relayers](https://blog.0xproject.com/18-ideas-for-0x-relayers-in-2018-80a1498b955f)
[Dharma Relayer](https://relayer.dharma.io/)

### Distributed Storage with Ipfs
[IPFS](https://ipfs.io/)

### Protocols
+ [0x](https://0xproject.com/)
+ [Dharma](https://dharma.io/)

### Resources
* [Program the Blockchain](https://programtheblockchain.com/)
* [Crypto Zombies](https://cryptozombies.io/)
* [Zastrin](https://www.zastrin.com/)
* [Dapp University](http://www.dappuniversity.com/)
* [Beginners Guide to Blockchain Programming](https://hackernoon.com/a-beginners-guide-to-blockchain-programming-4913d16eae31)


### Glossary: Blockchain
+ See also [https://bitcoin.org/en/vocabulary](https://bitcoin.org/en/vocabulary)

* **Address**: an address is essentially the representation of a public key belonging to a particular user; for example, the address associated with the private key given above is cd2a3d9f938e13cd947ec05abc7fe734df8dd826. Note that in practice, the address is technically the hash of a public key, but for simplicity it's better to ignore this distinction.

* **Transaction**: a transaction is a document authorizing some particular action associated with the blockchain. In a currency, the dominant transaction type is sending currency units or tokens to someone else; in other systems actions like registering domain names, making and fulfilling trade offers and entering into contracts are also valid transaction types.

* **Block**: a block is a package of data that contains zero or more transactions, the hash of the previous block ("parent"), and optionally other data. The total set of blocks, with every block except for the initial "genesis block" containing the hash of its parent, is called the blockchain and contains the entire transaction history of a network. Note that some blockchain-based cryptocurrencies instead use the word "ledger" for a blockchain; the two are roughly equivalent, although in systems that use the term "ledger" each block generally contains a full copy of the current state (eg. currency balances, partially fulfilled contracts, registrations) of every account allowing users to discard outdated historical data.

* **Account**: an account is the entry in a ledger, indexed by its address, that contains the complete data about the state of that account. In a currency system, this involves currency balances and perhaps unfulfilled trade orders; in other cases more complex relationships may be stored inside of accounts.

* **Proof of work**: one important property of a block in Bitcoin, Ethereum and many other crypto-ledgers is that the hash of the block must be smaller than some target value. The reason this is necessary is that in a decentralized system anyone can produce blocks, so in order to prevent the network from being flooded with blocks, and to provide a way of measuring how much consensus there is behind a particular version of the blockchain, it must in some way be hard to produce a block. Because hashes are pseudorandom, finding a block whose hash is less than 0000000100000000000000000000000000000000000000000000000000000000 takes an average of 4.3 billion attempts. In all such systems, the target value self-adjusts so that on average one node in the network finds a block every N minutes (eg. N = 10 for Bitcoin and 1 for Ethereum).

* **Nonce**: a meaningless value in a block which can be adjusted in order to try to satisfy the proof of work condition. Helps with avoiding Double Spends.

* **Mining**: mining is the process of repeatedly aggregating transactions, constructing a block and trying difference nonces until a nonce is found that satisfies the proof of work condition. If a miner gets lucky and produces a valid block, they are granted a certain number of coins as a reward as well as all of the transaction fees in the block, and all miners start trying to create a new block containing the hash of the newly generated block as their parent.

* **Stale**: a stale is a block that is created when there is already another block with the same parent out there; stales typically get discarded and are wasted effort.

* **Fork**: a situation where two blocks are generated pointing to the same block as their parent, and some portion of miners see one block first and some see the other. This may lead to two blockchains growing at the same time. Generally, it is mathematically near-certain that a fork will resolve itself within four blocks as miners on one chain will eventually get lucky and that chain will grow longer and all miners switch to it; however, forks may last longer if miners disagree on whether or not a particular block is valid.

* **Double spend**: a deliberate fork, where a user with a large amount of mining power sends a transaction to purchase some produce, then after receiving the product creates another transaction sending the same coins to themselves. The attacker then creates a block, at the same level as the block containing the original transaction but containing the second transaction instead, and starts mining on the fork. If the attacker has more than 50% of all mining power, the double spend is guaranteed to succeed eventually at any block depth. Below 50%, there is some probability of success, but it is usually only substantial at a depth up to about 2-5; for this reason, most cryptocurrency exchanges, gambling sites and financial services wait until six blocks have been produced ("six confirmations") before accepting a payment.

* **SPV client (or light client)** - a client that downloads only a small part of the blockchain, allowing users of low-power or low-storage hardware like smartphones and laptops to maintain almost the same guarantee of security by sometimes selectively downloading small parts of the state without needing to spend megabytes of bandwidth and gigabytes of storage on full blockchain validation and maintennance.

### Glossary: Ethereum Blockchain
+ See also: [http://ethereum.org/ethereum.html] (http://ethereum.org/ethereum.html)

* **Serialization**: the process of converting a data structure into a sequence of bytes. Ethereum internally uses an encoding format called recursive-length prefix encoding (RLP), described here

* **Patricia tree (or trie)**: a data structure which stores the state of every account. The trie is built by starting from each individual node, then splitting the nodes into groups of up to 16 and hashing each group, then making hashes of hashes and so forth until there is one final "root hash" for the entire trie. The trie has the important properties that (1) there is exactly one possible trie and therefore one possible root hash for each set of data, (2) it is very easy to update, add or remove nodes in the trie and generate the new root hash, (3) there is no way to modify any part of the tree without changing the root hash, so if the root hash is included in a signed document or a valid block the signature or proof of work secures the entire tree, and (4) one can provide just the "branch" of a tree going down to a particular node as cryptographic proof that that node is indeed in the tree with that exact content. Patricia trees are also used to store the internal storage of accounts as well as transactions and uncles. See here for a more detailed description.

* **GHOST**: GHOST is a protocol by which blocks can contain a hash of not just their parent, but also hashes for stales that are other children of the parent's parent (called uncles). This ensures that stales still contribute to blockchain security, and mitigates a problem with fast blockchains that large miners have an advantage because they hear about their own blocks immediately and so are less likely to generate stales.

* **Uncle**: a child of a parent of a parent of a block that is not the parent, or more generally a child of an ancestor that is not an ancestor. If A is an uncle of B, B is a nephew of A.

* **Account nonce**: a transaction counter in each account. This prevents replay attacks where a transaction sending eg. 20 coins from A to B can be replayed by B over and over to continually drain A's balance.

* **EVM code**: Ethereum virtual machine code, the programming language in which accounts on the Ethereum blockchain can contain code. The EVM code associated with an account is executed every time a message is sent to that account, and has the ability to read/write storage and itself send messages.

* **Message**: a sort of "virtual transaction" sent by EVM code from one account to another. Note that "transactions" and "messages" in Ethereum are different; a "transaction" in Ethereum parlance specifically refers to a physical digitally signed piece of data that goes in the blockchain, and every transaction triggers an associated message, but messages can also be sent by EVM code, in which case they are never represented in data anywhere.

* **Storage**: a key/value database contained in each account, where keys and values are both 32-byte strings but can otherwise contain anything.

* **Externally owned account**: an account controlled by a private key. Externally owned accounts cannot contain EVM code.

* **Contract**: an account which contains, and is controlled by, EVM code. Contracts cannot be controlled by private keys directly; unless built into the EVM code, a contract has no owner once released.

* **Ether**: the primary internal cryptographic token of the Ethereum network. Ether is used to pay transaction and computation fees for Ethereum transactions.

* **Gas**: a measurement roughly equivalent to computational steps. Every transaction is required to include a gas limit and a fee that it is willing to pay per gas; miners have the choice of including the transaction and collecting the fee or not. If the total number of gas used by the computation spawned by the transaction, including the original message and any sub-messages that may be triggered, is less than or equal to the gas limit, then the transaction processes. If the total gas exceeds the gas limit, then all changes are reverted, except that the transaction is still valid and the fee can still be collected by the miner. Every operation has a gas expenditure; for most operations it is 1, although some expensive operations fave expenditures up to 100 and a transaction itself has an expenditure of 500.

### Glossary: Cryptography
* **Computational infeasibility**: a process is computationally infeasible if it would take an impracticably long time (eg. billions of years) to do it for anyone who might conceivably have an interest in carrying it out. Generally, 280 computational steps is considered the lower bound for computational infeasibility.

* **Hash**: a hash function (or hash algorithm) is a process by which a document (ie. a piece of data or file) is processed into a small piece of data (usually 32 bytes) which looks completely random, and from which no meaningful data can be recovered about the document, but which has the important property that the result of hashing one particular document is always the same. Additionally, it is crucially important that it is computationally infeasible to find two documents that have the same hash. Generally, changing even one letter in a document will completely randomize the hash; for example, the SHA3 hash of "Saturday" is c38bbc8e93c09f6ed3fe39b5135da91ad1a99d397ef16948606cdcbd14929f9d, whereas the SHA3 hash of Caturday is b4013c0eed56d5a0b448b02ec1d10dd18c1b3832068fbbdc65b98fa9b14b6dbf. Hashes are usually used as a way of creating a globally agreed-upon identifier for a particular document that cannot be forged.

* **Encryption**: encryption is a process by which a document (plaintext) is combined with a shorter string of data, called a key (eg. c85ef7d79691fe79573b1a7064c19c1a9819ebdbd1faaab1a8ec92344438aaf4), to produce an output (ciphertext) which can be "decrypted" back into the original plaintext by someone else who has the key, but which is incomprehensible and computationally infeasible to decrypt for anyone who does not have the key.

* **Public key encryption**: a special kind of encryption where there is a process for generating two keys at the same time (typically called a private key and a public key), such that documents encrypted using one key can be decrypted with the other. Generally, as suggested by the name, individuals publish their public keys and keep their private keys to themselves.

* **Digital signature**: a digital signing algorithm is a process by which a user can produce a short string of data called a "signature" of a document using a private key such that anyone with the corresponding public key, the signature and the document can verify that (1) the document was "signed" by the owner of that particular private key, and (2) the document was not changed after it was signed. Note that this differs from traditional signatures where you can scribble extra text onto a document after you sign it and there's no way to tell the difference; in a digital signature any change to the document will render the signature invalid.

© Copyright 2018 MIT License

