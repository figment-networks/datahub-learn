---
description: Learn how to create, mint ,and transfer an NFT on Avalanche
---

# 1. Create an NFT using Avalanche wallet

## Introduction

Avalanche is a global financial network for the issuance and trade of digital goods. On Avalanche, these digital goods are represented as tokens, which can be assets or utilities. Some tokens are **fungible**, which means that one token is interchangeable for any other token. Real-world currency is fungible, for example; one $5 note is treated as being the same as any other $5 note but let's say for AVAX the value is not constant it changes as its financial network grows.

Avalanche also supports non-fungible tokens \(NFTs\). By definition, each NFT is unique and not perfectly interchangeable for any other NFT. For example, there could be an NFT that represents ownership of a real-world piece of art; each piece of art, like each NFT, is unique. NFTs represent digital scarcity and may prove to have even greater utility than traditional fungible tokens.

# Create NFTs with the Avalanche Wallet

\*\*\*\*[**The original tutorial can be found in the AVA Labs documentation here**](https://docs.avax.network/build/tutorials/smart-digital-assets/wallet-nft-studio). 

## Non-Fungible Tokens on Avalanche

Besides the native AVAX token, [Avalanche platform](https://docs.avax.network/build/tutorials/platform) natively supports the creation of other types of digital assets: fixed-cap assets, variable-cap assets, and non-fungible tokens \(NFTs\).

As opposed to regular tokens, which are interchangeable \(fungible\), meaning that each one is the same, each non-fungible token is unique on the network, with a distinct ID making it different from any other. This enables many use cases that would be impossible with interchangeable tokens, like proof of ownership of a unique asset.

## NFT Studio on Avalanche Wallet

To make experimenting with the creation and exchange of NFTs easier, we have built **NFT Studio** into the [Avalanche Wallet](https://wallet.avax.network/), where you can use it to create NFTs as assets that we call Collectibles. Collectibles can be generic NFTs with a picture and a description, or custom NFTs with payloads containing JSON, custom URL, or UTF-8 data. You can create them using a simple point and click interface, enabling you to go from the idea of sending NFTs to your friends within minutes. No technical knowledge is required.

To access the **NFT Studio**, log into your Avalanche Wallet, and on the left side select **Studio**:

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/create-new-wallet.png)

This will open the NFT Studio. There you have two options: **New Family**, for the creation of a new family of NFTs, and **Mint Collectible** for creating new assets in existing families. We need to create our first family of NFTs, so click **New Family**.

### Create NFT Family

There you will be asked to enter the name of your collectible family, as well as a symbol \(ticker\). Names do not have to be unique.
When you click on the Access button, you will be directed to a new window "Generate key Phrase", where you will see a list of keycodes and need to save or capture the given set of 12-16 words somewhere safe.
This list of code will help to access your account( do not share it with anyone)

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/generate-new-key.png)

you will be directed to your key phrase window,

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/your-key-phrase.png)

After Generating and capturing the list of codes, you will again need to verify the key-phrase( list of codes), where you have to fill in the blanks in order to access your account.

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/verify-key-phrase.png)

Now, you can access your account. Click access and you will be directed to the account window. 

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/access-wallet.png)

In your avalanche wallet window, you will see-
1. Balance
2. Assets
3. Wallet Address
4. Transaction & Transaction History

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/avalanche-wallet.png)
On the right sidebar, you will have other options too, like
1. Send
2. Earn
3. Cross-chain
5. Studio
6. Activity
7. Manage keys 
8. Advanced

### Mint NFTs

Before Minting, Go to the Studio tab as shown below- you will see a heading " Collectibles", in which two sections are provided 
1. New family 
2. Mint collectibles

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/avalanche-studio.PNG)

concerning the above image, go and click on Studio, where you have to create new family if you are a new user.

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/create-new-family-studio.png)

When you decided upon the name, ticker, and a number of groups, press **Create** to create the collectible family. The transaction fee will be deducted from your wallet's balance. When the family is created, you will see the transaction ID \(TxID\), as well as parameters for the family. You can use the TxID to look up the transaction in [the explorer](https://explorer.avax.network/)
Besides the name and the ticker, you will need to enter **Number of Groups**, that is, how many distinct collectibles will the newly created family hold. Choose carefully, because once created, the parameters of the collectible family cannot be changed.

After pressing **Mint Collectible** you will be presented with a list of all the Collectible families that still have Collectible groups that have not been created yet
as shown in the above image.
you need to create a new mint, by providing a name, a code, and a number of groups. 
Note- (a fee of 0.1 Avax will be deducted from your balance), to give it a test, follow below steps -  
to get Avax coin, you need to change the network setting from mainnet to Fuji-test network.

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/change-network-to-fuji.png)

Go to Avax faucet-

![](https://faucet.avax-test.network/)

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/avax-faucet-testnet.PNG)

copy the address from your avalanche wallet and paste it in the address bar, you will receive 2 Avax coins. 

Select the family we have just created. You will be presented with a form to fill out with the parameters of the new collectible:

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/mint-collectibles.png)

Press **Back to Studio** to return, and we're ready to create our first collectibles. Press **Mint Collectible**.
but if minting is not supported, then you need to create a new collectible family as shown below. 

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/wallet-avax-network.png)

By default, a **Generic** type of collectible will be selected. That is an NFT that has a **Title**, **URL** for the image, and a **Description**. Enter the required data, as well as the **Quantity**, which will determine how many copies of the collectible will be created, and therefore, how many of them you will be able to send. As before, enter the data carefully, you won't be able to change anything once collectibles are minted. You will see a preview of the data where you can check how your collectible will look like.

If you would like to have something else besides a picture collectible, select **Custom**.

![](../../../.gitbook/assets/nft-studio-05-custom.png)

A custom collectible can contain an **UTF-8** encoded string, an **URL**, or a **JSON** payload. The size of the data cannot exceed 1024 characters.

After you enter and check the data, press **Mint** to create the collectible. Transaction fees will be deducted from your wallet, and a newly created collectible will be placed in your wallet.

### See your collectibles

An overview of your collectibles is always visible at the top of the screen, along with your balances.

![](../../../.gitbook/assets/nft-studio-06-overview.png)

To see your collectibles in more detail, select **Portfolio** from the left-hand side menu. You will be presented with a screen showing all of your assets, with tokens selected by default. Change the selection to **Collectibles** by clicking the corresponding tab.

![](../../../.gitbook/assets/nft-studio-07-collectibles.png)

For each Generic collectible, a picture will be shown, along with the title, and the number indicating how many copies of the collectible are in your portfolio. Hovering over the collectible with your pointer will show the detailed description:

![](../../../.gitbook/assets/nft-studio-08-detail.png)

If you select a collectible by clicking on it, you will see which group it belongs to, its quantity, along with the **Send** button.

## Send & Transfer NFTs

To send your collectible to someone, either click the **Send** button on the selected collectible in the Portfolio, or navigate to the **Send** tab on the left-hand side menu, and click **Add Collectible**:

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/send-nft-different-address.png)

You will be presented with a menu to select a collectible you wish to send.

![](../../../.gitbook/assets/nft-studio-10-multiple.png)

You can send multiple collectibles in a single transaction. Clicking the label on the collectible will let you edit the number of copies you wish to send. You can send multiple families and collectible types in a single transaction.

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/confirm-transcation.png)

When you have entered the destination address, and optionally entered the memo text, press **Confirm** to initiate the transaction.

![](https://github.com/Devilla/datahub-learn/blob/master/.gitbook/assets/transaction-history.png)

After pressing **Send Transaction** it will be published on the network, and the transaction fee will be deducted from your balance. Collectibles will be deposited into the destination address shortly after.

## Summary

Now, you should know how to create NFT families, mint NFT groups, and send them to other addresses. Have fun with it! If you would like to know the technical background of how NFTs work on the Avalanche network or would like to build products using NFTs, please check out the [NFT tutorial](https://learn.figment.io/network-documentation/avalanche/tutorials/creating-an-nft-part-1). 

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!
