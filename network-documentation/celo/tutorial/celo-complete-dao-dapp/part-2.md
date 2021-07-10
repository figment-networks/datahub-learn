---
title: Build a Decentralized Autonomous Organization (DAO) on Celo Part 2
description: Learn how to build a fully functional DAO by writing the Smart Contract Code and then build a React Native App to communicate with the Smart Contract
---
# Build a Decentralized Autonomous Organization (DAO) on Celo - Part 2

In this tutorial, we are going to build a fully functional DAO by writing the Smart Contract Code and then build a React Native App to communicate with the Smart Contract.
This part 2 section focuses on Writing the Smart Contract Code

&nbsp;

# Prerequisite

[Solidity v8](https://docs.soliditylang.org/en/v0.8.0/installing-solidity.html)

&nbsp;

# Coding the DAO Smart Contract
The smart contract is built using the `Solidity` language. 
Initial Setup 
The first thing to do here is to initialize a basic solidity project using truffle. Follow the process here to install truffle if you don’t have that previously installed. Next up is running `truffle init` in our preferred location to install the project. After running this command, truffle will set up a basic solidity project with the necessary files.
Code
Here is the full code for the smart contract, we’ll go over the different functions here to see what they do and why they are needed.

```code
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract CharloDAO is ReentrancyGuard, AccessControl {
    bytes32 public constant CONTRIBUTOR_ROLE = keccak256("CONTRIBUTOR");
    bytes32 public constant STAKEHOLDER_ROLE = keccak256("STAKEHOLDER");
    uint32 constant minimumVotingPeriod = 1 weeks;
    uint256 numOfProposals;

    struct CharityProposal {
        uint256 id;
        uint256 amount;
        uint256 livePeriod;
        uint256 votesFor;
        uint256 votesAgainst;
        string description;
        bool votingPassed;
        bool paid;
        address payable charityAddress;
        address proposer;
        address paidBy;
    }

    mapping(uint256 => CharityProposal) private charityProposals;
    mapping(address => uint256[]) private stakeholderVotes;
    mapping(address => uint256) private contributors;
    mapping(address => uint256) private stakeholders;

    event ContributionReceived(address indexed fromAddress, uint256 amount);
    event NewCharityProposal(address indexed proposer, uint256 amount);
    event PaymentTransfered(
        address indexed stakeholder,
        address indexed charityAddress,
        uint256 amount
    );

    modifier onlyStakeholder(string memory message) {
        require(hasRole(STAKEHOLDER_ROLE, msg.sender), message);
        _;
    }

    modifier onlyContributor(string memory message) {
        require(hasRole(CONTRIBUTOR_ROLE, msg.sender), message);
        _;
    }

    function createProposal(
        string calldata description,
        address charityAddress,
        uint256 amount
    )
        external
        onlyStakeholder("Only stakeholders are allowed to create proposals")
    {
        uint256 proposalId = numOfProposals++;
        CharityProposal storage proposal = charityProposals[proposalId];
        proposal.id = proposalId;
        proposal.proposer = payable(msg.sender);
        proposal.description = description;
        proposal.charityAddress = payable(charityAddress);
        proposal.amount = amount;
        proposal.livePeriod = block.timestamp + minimumVotingPeriod;

        emit NewCharityProposal(msg.sender, amount);
    }

    function vote(uint256 proposalId, bool supportProposal)
        external
        onlyStakeholder("Only stakeholders are allowed to vote")
    {
        CharityProposal storage charityProposal = charityProposals[proposalId];

        votable(charityProposal);

        if (supportProposal) charityProposal.votesFor++;
        else charityProposal.votesAgainst++;

        stakeholderVotes[msg.sender].push(charityProposal.id);
    }

    function votable(CharityProposal storage charityProposal) private {
        if (
            charityProposal.votingPassed ||
            charityProposal.livePeriod <= block.timestamp
        ) {
            charityProposal.votingPassed = true;
            revert("Voting period has passed on this proposal");
        }

        uint256[] memory tempVotes = stakeholderVotes[msg.sender];
        for (uint256 votes = 0; votes < tempVotes.length; votes++) {
            if (charityProposal.id == tempVotes[votes])
                revert("This stakeholder already voted on this proposal");
        }
    }

    function payCharity(uint256 proposalId)
        external
        onlyStakeholder("Only stakeholders are allowed to make payments")
    {
        CharityProposal storage charityProposal = charityProposals[proposalId];

        if (charityProposal.paid)
            revert("Payment has been made to this charity");

        if (charityProposal.votesFor <= charityProposal.votesAgainst)
            revert(
                "The proposal does not have the required amount of votes to pass"
            );

        charityProposal.paid = true;
        charityProposal.paidBy = msg.sender;

        emit PaymentTransfered(
            msg.sender,
            charityProposal.charityAddress,
            charityProposal.amount
        );

        return charityProposal.charityAddress.transfer(charityProposal.amount);
    }

    receive() external payable {
        emit ContributionReceived(msg.sender, msg.value);
    }

    function makeStakeholder(uint256 amount) external {
        address account = msg.sender;
        uint256 amountContributed = amount;
        if (!hasRole(STAKEHOLDER_ROLE, account)) {
            uint256 totalContributed =
                contributors[account] + amountContributed;
            if (totalContributed >= 5 ether) {
                stakeholders[account] = totalContributed;
                contributors[account] += amountContributed;
                _setupRole(STAKEHOLDER_ROLE, account);
                _setupRole(CONTRIBUTOR_ROLE, account);
            } else {
                contributors[account] += amountContributed;
                _setupRole(CONTRIBUTOR_ROLE, account);
            }
        } else {
            contributors[account] += amountContributed;
            stakeholders[account] += amountContributed;
        }
    }

    function getProposals()
        public
        view
        returns (CharityProposal[] memory props)
    {
        props = new CharityProposal[](numOfProposals);

        for (uint256 index = 0; index < numOfProposals; index++) {
            props[index] = charityProposals[index];
        }
    }

    function getProposal(uint256 proposalId)
        public
        view
        returns (CharityProposal memory)
    {
        return charityProposals[proposalId];
    }

    function getStakeholderVotes()
        public
        view
        onlyStakeholder("User is not a stakeholder")
        returns (uint256[] memory)
    {
        return stakeholderVotes[msg.sender];
    }

    function getStakeholderBalance()
        public
        view
        onlyStakeholder("User is not a stakeholder")
        returns (uint256)
    {
        return stakeholders[msg.sender];
    }

    function isStakeholder() public view returns (bool) {
        return stakeholders[msg.sender] > 0;
    }

    function getContributorBalance()
        public
        view
        onlyContributor("User is not a contributor")
        returns (uint256)
    {
        return contributors[msg.sender];
    }

    function isContributor() public view returns (bool) {
        return contributors[msg.sender] > 0;
    }
}
```
&nbsp;

Let’s go ahead and dissect the different functions that make up the smart contract.

&nbsp;

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
```
The first line here is simply the licence declaration that every solidity contract should start with: according to the Solidity [docs](https://docs.soliditylang.org/en/v0.8.4/layout-of-source-files.html). Line two is the pragma directive to enable certain compiler features or checks.

&nbsp;
```
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
```
We’ll be relying on [OpenZepellin's](https://openzeppelin.com/) ReentrancyGuard and AccessControl here so we should go ahead and import those.

&nbsp;
```
bytes32 public constant CONTRIBUTOR_ROLE = keccak256("CONTRIBUTOR");
bytes32 public constant STAKEHOLDER_ROLE = keccak256("STAKEHOLDER");
```
User’s of the DAO will be of two types - Contributors and Stakeholders - we need to declare two constants here.. We’ll eventually use them to register and differentiate users.

&nbsp;
```
uint32 constant minimumVotingPeriod = 1 weeks;
```
The `minimumVotingPeriod` variable holds the number of days a proposal can be voted on in UNIX time. The global variable `weeks` is a suffix provided by Solidity, `1 weeks` here translates to the total seconds of UNIX time a week from now.

&nbsp;
```
uint256 numOfProposals;
```
This variable is incremented every time a new charity proposal is added. This is so we could iterate through the charity proposals in the mapping data type as Solidity doesn’t provide a way to step through mappings.

&nbsp;
```
    struct CharityProposal {
        uint256 id;
        uint256 amount;
        uint256 livePeriod;
        uint256 votesFor;
        uint256 votesAgainst;
        string description;
        bool votingPassed;
        bool paid;
        address payable charityAddress;
        address proposer;
        address paidBy;
    }
```
This here is the Proposal type definition. This will hold the necessary data that makes up our proposal object.

&nbsp;
```
    mapping(uint256 => CharityProposal) private charityProposals;
    mapping(address => uint256[]) private stakeholderVotes;
    mapping(address => uint256) private contributors;
    mapping(address => uint256) private stakeholders;
```

&nbsp;

A group of four mapping type:
`charityProposals` has uint256 and CharityProposal types as key and value respectively. This holds the list of Proposals in the DAO. It uses the id of the Proposal as key and the Proposal itself as the value.
`stakeholderVotes` holds an `id` only list of the Proposals a particular stakeholder has voted on.
`contributors` has the balance of a contributor as the value and the address of the contributor as the key to index into the mapping.
`stakeholders` has the balance of a stakeholder as the value and the address of the stakeholder as the key to index into the mapping.

```
    event ContributionReceived(address indexed fromAddress, uint256 amount);
    event NewCharityProposal(address indexed proposer, uint256 amount);
    event PaymentTransfered(
        address indexed stakeholder,
        address indexed charityAddress,
        uint256 amount
    );
```

Three events are emitted for logging purposes every time there’s a new Proposal, new contribution and new payment transfer

```
    modifier onlyStakeholder(string memory message) {
        require(hasRole(STAKEHOLDER_ROLE, msg.sender), message);
        _;
    }
```
```
    modifier onlyContributor(string memory message) {
        require(hasRole(CONTRIBUTOR_ROLE, msg.sender), message);
        _;
    }
```
These modifiers will be used to control who has access to specific functions
```
    function createProposal(
        string calldata description,
        address charityAddress,
        uint256 amount
    )
        external
        onlyStakeholder("Only stakeholders are allowed to create proposals")
    {
        uint256 proposalId = numOfProposals++;
        CharityProposal storage proposal = charityProposals[proposalId];
        proposal.id = proposalId;
        proposal.proposer = payable(msg.sender);
        proposal.description = description;
        proposal.charityAddress = payable(charityAddress);
        proposal.amount = amount;
        proposal.livePeriod = block.timestamp + minimumVotingPeriod;


        emit NewCharityProposal(msg.sender, amount);
    }
```
An external function that we can call in our dApp to add a new Proposal, it declares three parameters: `description`, `charityAddress` and `amount`.
Notice the `onlyStakeholder` modifier in use here, this is needed to restrict this function only to stakeholders. On line 43, we assign the value of the `numOfProposal` variable we declared earlier to `proposalId` then incremented `numOfProposals`. On line 44, we declared a proposal variable of the CharityProposal type then assigns its reference to one of the buckets in the `charityProposals` mapping. Line 45-50 assigns the necessary values to their counterpart in the CharityProposal object. Lastly, the `NewCharityProposal` event is emitted on line 52.
```
    function vote(uint256 proposalId, bool supportProposal)
        external
        onlyStakeholder("Only stakeholders are allowed to vote")
    {
        CharityProposal storage charityProposal = charityProposals[proposalId];


        votable(charityProposal);


        if (supportProposal) charityProposal.votesFor++;
        else charityProposal.votesAgainst++;


        stakeholderVotes[msg.sender].push(charityProposal.id);
    }
```
The `vote()` function is an external function that allows voting on proposals when called with the proposal’s id and a second true/false argument depending on whether the vote is in support or against the proposal.

```
    function votable(CharityProposal storage charityProposal) private {
        if (
            charityProposal.votingPassed ||
            charityProposal.livePeriod <= block.timestamp
        ) {
            charityProposal.votingPassed = true;
            revert("Voting period has passed on this proposal");
        }


        uint256[] memory tempVotes = stakeholderVotes[msg.sender];
        for (uint256 votes = 0; votes < tempVotes.length; votes++) {
            if (charityProposal.id == tempVotes[votes])
                revert("This stakeholder already voted on this proposal");
        }
    }
```
`votable()` is called in the `vote()` function. It is used to verify if a proposal can be voted on.
```
    function payCharity(uint256 proposalId)
        external
        onlyStakeholder("Only stakeholders are allowed to make payments")
    {
        CharityProposal storage charityProposal = charityProposals[proposalId];


        if (charityProposal.paid)
            revert("Payment has been made to this charity");


        if (charityProposal.votesFor <= charityProposal.votesAgainst)
            revert(
                "The proposal does not have the required amount of votes to pass"
            );


        charityProposal.paid = true;
        charityProposal.paidBy = msg.sender;


        emit PaymentTransfered(
            msg.sender,
            charityProposal.charityAddress,
            charityProposal.amount
        );


        return charityProposal.charityAddress.transfer(charityProposal.amount);
}
```
This function makes payment to a Charity after the voting period of the proposal has ended. It takes the proposal id as an argument and gets the proposal from the mapping. Line 95-101 checks if the charity has already been paid or if the number of votes in support of the proposal is lesser than those against. If those two conditions are true then the transaction will be reverted with the right error message. If not, make the paid property of the proposal true, set the address of the stakeholder making the payment, emit the PaymentTransfered event for logging purposes then finally transfer the payment to the charity.

```
receive() external payable {
        emit ContributionReceived(msg.sender, msg.value);
}
```

This is needed so the contract can receive contributions without throwing an error. The function emits the ContributionReceived event previously declared in the upper scope.

```
function makeStakeholder(uint256 amount) external {
        address account = msg.sender;
        uint256 amountContributed = amount;
        if (!hasRole(STAKEHOLDER_ROLE, account)) {
            uint256 totalContributed =
                contributors[account] + amountContributed;
            if (totalContributed >= 5 ether) {
                stakeholders[account] = totalContributed;
                contributors[account] += amountContributed;
                _setupRole(STAKEHOLDER_ROLE, account);
                _setupRole(CONTRIBUTOR_ROLE, account);
            } else {
                contributors[account] += amountContributed;
                _setupRole(CONTRIBUTOR_ROLE, account);
            }
        } else {
            contributors[account] += amountContributed;
            stakeholders[account] += amountContributed;
        }
}
```

This function adds a new Stakeholder to the DAO if the total contribution of the user is more than or equal to 5 Celo. If the total contribution is less than 5 then the user is made a Contributor instead.

```
function getProposals()
        public
        view
        returns (CharityProposal[] memory props)
{
        props = new CharityProposal[](numOfProposals);


        for (uint256 index = 0; index < numOfProposals; index++) {
            props[index] = charityProposals[index];
        }
}
```

We are returning a list of all the proposals in the DAO here. Solidity doesn’t have iterators for the mapping type so we declare a fixed-size array, used the `numOfProposals` variable as the upper limit of our loop. For each iteration, we assign the proposal at the current index to the index in our fixed-size array then return the array. Essentially, this fetches our proposals and returns them as an array.

```
function getProposal(uint256 proposalId)
        public
        view
        returns (CharityProposal memory)
{
    return charityProposals[proposalId];
}
```

The `getProposal()` function takes a proposal id as an argument get the proposal from the proposal mapping then return the proposal.

```
function getStakeholderVotes()
        public
        view
        onlyStakeholder("User is not a stakeholder")
        returns (uint256[] memory)
{
    return stakeholderVotes[msg.sender];
}
```

The `getStakeholderVotes()` gets and returns a list containing the id of all the proposals that a particular stakeholder has voted on.

```
function getStakeholderBalance()
        public
        view
        onlyStakeholder("User is not a stakeholder")
        returns (uint256)
{
    return stakeholders[msg.sender];
}
```

As the name suggests, the `getStakeholderBalance()` functions return the total amount of contribution a stakeholder has contributed to the DAO.

```
function isStakeholder() public view returns (bool) {
    return stakeholders[msg.sender] > 0;
}
```

The `isStakeholder()` function returns true/false depending on whether the caller is a stakeholder or not

```
function getContributorBalance()
        public
        view
        onlyContributor("User is not a contributor")
        returns (uint256)
{
    return contributors[msg.sender];
}
```

As the name suggests, this function returns the total balance of a contributor.

```
function isContributor() public view returns (bool) {
    return contributors[msg.sender] > 0;
}
```

The `isContributor()` function returns true/false depending on whether the caller is a contributor or not

&nbsp;

**Now that we have written the Smart Contract Code, the next step is to build the React Native App.**

[Part 3](./part-3)
