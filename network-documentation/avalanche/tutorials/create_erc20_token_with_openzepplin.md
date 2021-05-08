# Overview 

## Creating ERC20 token with OpenZeppelin 🥇

## About the author 

Hi, I'm Laksh, aged 16 and a passionate Developer 🚀 from [India](https://en.wikipedia.org/wiki/India) with a strong interest in Coding and Developing new things.

🌱 I’m just a beginner Who knows Web Development and knows  HTML , CSS , JS , Python and C. I too have interest in Cyber Security and  have  a lot of certifications in various fields.

🥇 On the way Of learning More... 

📫 How to reach me:

Follow me on [Instagram](https://www.instagram.com/hackerlaksh22/) 🐦

Shoot me a [mail](hackersarenayt22@gmail.com) 💌

Ping me on [Telegram](https://t.me/Laksh_Coder) ↗️

## What is OpenZeppelin ?

# <img src="https://kajabi-storefronts-production.global.ssl.fastly.net/kajabi-storefronts-production/blogs/19054/images/hlzHh5QYTeKpf8vCUbeQ_logo_openzeppelin_b8c833659d043cf69a7dd13d3487defdac2af8fd9d487db68e457f129284f8dc_opti.png" alt="OpenZeppelin" height="150px">

**A library for secure smart contract development.** Build on a solid foundation of community-vetted code.

 * Implementations of standards like [ERC20](https://docs.openzeppelin.com/contracts/erc20) and [ERC721](https://docs.openzeppelin.com/contracts/erc721).
 * Flexible [role-based permissioning](https://docs.openzeppelin.com/contracts/access-control) scheme.
 * Reusable [Solidity components](https://docs.openzeppelin.com/contracts/utilities) to build custom contracts and complex decentralized systems.
 * First-class integration with the [Gas Station Network](https://docs.openzeppelin.com/contracts/gsn) for systems with no gas fees!
 * [Audited](https://github.com/OpenZeppelin/openzeppelin-contracts/tree/master/audit) by leading security firms (_last full audit on v2.0.0_).


## Installation 🔰

Firstly we have to install [OpenZeppelin](https://openzeppelin.com). Execute this command for installation.

```console
$ npm install @openzeppelin/contracts
```
If you are upgrading OpenZeppelin in future , it will not affect the contracts because 
OpenZeppelin Contracts features a [stable API](https://docs.openzeppelin.com/contracts/releases-stability#api-stability), which means your contracts won’t break unexpectedly when upgrading to a newer minor version.

### Usage 💻

Once installed, you can use the contracts in the library by importing them:
Import by this code below 

```solidity
// contracts/MyNFT.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
contract MyNFT is ERC721 {
constructor() ERC721("MyNFT", "MNFT") {
    }
}
```

_If you're new to smart contract development, head to [Developing Smart Contracts](https://docs.openzeppelin.com/learn/developing-smart-contracts) to learn about creating a new project and compiling your contracts._

* To keep your system secure, you should always use the installed code as-it-is, and neither copy-paste it from online sources, nor modify it yourself.
* The library is designed so that only the contracts and functions you use are deployed, so you don’t need to worry about it needlessly increasing gas costs.
* In simple words don't blindly copy paste the code from  any third party site because it may affect your financial balance. 

Be safe.

Please report any security issues you find to security@openzeppelin.org.

## Learn More

The guides in the [docs site](https://docs.openzeppelin.com/contracts) will teach about different concepts, and how to use the related contracts that OpenZeppelin Contracts provides:

* [Access Control](https://docs.openzeppelin.com/contracts/access-control): decide who can perform each of the actions on your system.
* [Tokens](https://docs.openzeppelin.com/contracts/tokens): create tradeable assets or collectives, and distribute them via [Crowdsales](https://docs.openzeppelin.com/contracts/crowdsales).
* [Gas Station Network](https://docs.openzeppelin.com/contracts/gsn): let your users interact with your contracts without having to pay for gas themselves.
* [Utilities](https://docs.openzeppelin.com/contracts/utilities): generic useful tools, including non-overflowing math, signature verification, and trustless paying systems.

The [full API](https://docs.openzeppelin.com/contracts/api/token/ERC20) is also thoroughly documented, and serves as a great reference when developing your smart contract application. You can also ask for help or follow Contracts's development in the [community forum](https://forum.openzeppelin.com).

Finally, you may want to take a look at the [guides on our blog](https://blog.openzeppelin.com/guides), which cover several common use cases and good practices.. The following articles provide great background reading, though please note, some of the referenced tools have changed as the tooling in the ecosystem continues to rapidly evolve.

* [The Hitchhiker’s Guide to Smart Contracts in Ethereum](https://blog.openzeppelin.com/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05) will help you get an overview of the various tools available for smart contract development, and help you set up your environment.
* [A Gentle Introduction to Ethereum Programming, Part 1](https://blog.openzeppelin.com/a-gentle-introduction-to-ethereum-programming-part-1-783cc7796094) provides very useful information on an introductory level, including many basic concepts from the Ethereum platform.
* For a more in-depth dive, you may read the guide [Designing the Architecture for Your Ethereum Application](https://blog.openzeppelin.com/designing-the-architecture-for-your-ethereum-application-9cec086f8317), which discusses how to better structure your application and its relationship to the real world.

## Tokens ⬇️

 * Token is the most powerful and most misunderstood tool in blockchain.
 * A token is a representation of something in the blockchain. This is something that we call money, time, services, shares in a company, a virtual pet, anything. 
 * By representing things as tokens, we can allow smart contracts to interact with them, exchange them, create or destroy them.
 * Beginners just undderstand  tokens in form of money or services.

### Creating ERC20 Token ☣️

# <img src="https://www.technoloader.com/blog/wp-content/uploads/2019/06/ERC20-Token.jpg" alt="ERC20 Token" height="100%">

 * ERC20 token is the most convient token, but remmember all tokens are equal no price difference or speciality is there.
 * An ERC20 token contract keeps track of fungible tokens: any one token is exactly equal to any other token; no tokens have special rights or behavior associated with them. 
 * This makes ERC20 tokens useful for things like a medium of exchange currency, voting rights, staking, and more.

*ERC20 is the most widespread token standard for fungible assets, somewhat limited by its simplicity and used widely and is trusted wisely.*

### Constructing an ERC20 Token Contract 🎊

 * Now you will be wondering what is contract ? Don't worry we are here to help you.
 * Contracts helps you minimize risk by using battle-tested libraries of smart contracts for Ethereum and other blockchains. 
 * It includes the most used implementations of ERC standards. Using Contracts, we can easily create our own ERC20 token contract, which will be used to track Gold (GLD), an internal currency in a hypothetical game.

For more queries you can visit OpenZeppelin official repository:
# <img src="https://opengraph.githubassets.com/73f1f10a94df8dc52088c3e5a93938221212236e171197e52a094f59787f8b83/OpenZeppelin/openzeppelin-contracts" alt="ERC20 Token" height="100%">

Here’s what our GLD token might look like.

```solidity
    // contracts/GLDToken.sol
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;
    import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
    contract GLDToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("Gold", "GLD") {
        _mint(msg.sender, initialSupply);
        }
    }
```
 * Our contracts are often used via inheritance, and here we’re reusing ERC20 for both thebasic standard implementation and the name, symbol, and decimals optional extensions. 
Additionally, we’re creating an initialSupply of tokens, which will be assigned to the address that deploys the contract.


**Congrats 🥳 That’s it!** 

 * Once deployed, we will be able to query(find out) the deployer’s balance:

```solidity
    > GLDToken.balanceOf(deployerAddress)
    1000000000000000000000
```
 * We can also transfer these tokens to other accounts:

```solidity
    > GLDToken.transfer(otherAddress, 300000000000000000000)
    > GLDToken.balanceOf(otherAddress)
    300000000000000000000
    > GLDToken.balanceOf(deployerAddress)
    700000000000000000000
```

### Advantages of using OpenZeppelin 🌈

## The standard for secure blockchain applications

* OpenZeppelin provides security products to build, automate, and operate decentralized applications. 

* OpenZeppelin also protect leading organizations by performing security audits on their systems and products. 

* OpenZeppelin is Focused on security using top level standard contracts security patterns and best practices. 

* OpenZeppelin has modular approach which uses simple, robust code. Easy collaboration and auditing. 

* OpenZeppelin is open source which means Community-driven and is used by the biggest players in the industry.

Thank you for reading 
Have a great day 🥰

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

Made with ❤️ in [India](https://en.wikipedia.org/wiki/India).
