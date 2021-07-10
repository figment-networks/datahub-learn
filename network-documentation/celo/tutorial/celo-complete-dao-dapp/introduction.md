---
title: Build a Decentralized Autonomous Organization (DAO) on Celo.
description: Learn how to build a fully functional DAO by writing the Smart Contract Code and then build a React Native App to communicate with the Smart Contract
---

# Build a Decentralized Autonomous Organization (DAO) on Celo.

In this tutorial, we are going to build a fully functional DAO by writing the Smart Contract Code and then build a React Native App to communicate with the Smart Contract.

&nbsp;

# Introduction.

## What is a DAO? 

A [DAO](https://learn.figment.io/other/glossary#d) is represented as rules encoded as a computer program that is transparent, controlled by the organization members, and not influenced by a central government. It decides which decision will be made by a decentralized organization. 

In non-technical terms: **DAOs are an effective and safe way to work with like-minded folks around the globe**.

Think of DAOs like an internet-native business that’s collectively owned and managed by its members. They have built-in treasuries that no one has the authority to access without the approval of the group. Decisions are governed by proposals and voting to ensure everyone in the organization has a voice.

There’s no CEO who can authorize spending based on their own whims and no chance of a dodgy CFO manipulating the books. Everything is out in the open and the rules around spending are baked into the DAO via its code.

Read more [here](https://ethereum.org/en/dao/). 

**In this tutorial, you will learn how to build a complete Charity DAO. The Charity DAO is built using React Native dApp. The Smart Contract code is deployed on the Alfajores testnet. The contract allows the members to contribute to the DAO. Members can initiate/suggest charity proposals and stakeholders will have to vote to approve/reject the proposal within a stipulated period of time. After the elapse of the stipulated time, the DAO contract will disburse the funds pooled into the Charity DAO**.

&nbsp;

# Prerequisite

This article assumes that you have basic knowledge of Solidity, JavaScript (TypeScript), and how to start a React Native App using expo. It is also assumed that you have read the expo documentation and have basic knowledge of the Celo Wallet.

- [Celo Wallet](https://docs.celo.org/getting-started/alfajores-testnet/using-the-mobile-wallet)
- [React Native using expo](https://docs.expo.io/)
- [DappKit](https://docs.celo.org/developer-guide/dappkit/setup)
- [Redux](https://redux.js.org/introduction/getting-started)

&nbsp;

# Requirements
For this Tutorial, we will need the following installed:
- React Native
- NodeJS v12 and above
- Truffle
- The Alphajores Wallet Kindly fund the wallet using the Faucet

To aid your learning, this project is broken up into 3 parts: 
- Coding the DAO Smart Contract
- Building the React Native dApp
- Redux

&nbsp;

We will begin by outlining a visual representation of the dApp, its features, and the various screens used in interacting with the smart contract.

&nbsp;

![Diagram](./images/diagram.png)

&nbsp;

## Features
The App features include: 
- Users connect their Celo Wallet to join the Charity DAO
* Users contribute Celo to become Contributors of the DAO
- Contributors that have made 200 or more total contributions are automatically made Stakeholders.
* Only a stakeholder of the DAO can vote on proposals.
- Contributors and/or Stakeholders can create a new proposal.
* A newly created proposal has a date when voting on it closes.
- Stakeholders can upvote or downvote a proposal.
* Once the expiry date for voting on a Proposal closes, a stakeholder then pays out the requested amount to the Charity.

&nbsp;

This is the overview introduction to the project. The next sections are categorized into 3 parts
Writing the Smart Contract Code.
Building the React Native App.
Connecting the React Native App to the Smart Contract using Redux.

[Part 2](./part-2)