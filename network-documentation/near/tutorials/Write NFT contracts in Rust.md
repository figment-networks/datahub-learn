# NEAR NFT tutorial


## Introduction

[Non-Fungible Tokens] are unique records of ownership on the blockchain.  Usually an NFT is tied to something interesting and rare, such as an artwork, a concert ticket, a collectable cat, a domain name, or a real physical object.  NFTs can be bought, sold, given away, and can even be minted or destroyed, depending on the rules of the contract.  [Cryptokitties] and [SuperRare] are two popular examples of Ethereum-based NFTs. We can implement NFTs just as easily on NEAR.

In this tutorial we will issue a new species of NFT: the CryptoFlarn, a single-celled organism with unique DNA.  Our smart contract will allow Flarns to be created, collected and traded on the NEAR blockchain.

### About NFT standards:

There are many standards for NFTs!  However the most widely supported by far is the [ERC721] standard, which defines how NFTs can be created and transfered among other things. This standard works well, but like all the ERC standards it is defined only for the Ethereum blockchain.  ERC721 may be portable to NEAR once [NEAR EVM] emulation is available, but for now, the NEAR team has created an NFT reference implementation that uses a different NFT standard: [NEP-4], which is defined in a language-independent way that is more compatible with NEAR.  

NEP-4 is a very simple standard that does the bare minimum required to support ownership and transfer of NFTs, but it includes the possibility of delegating authority to other users or to other smart contracts.  This is a powerful feature, because it means that future enhancements might be added by cross-contract calls with another, smarter contract, instead of having to upgrade the contract we write today.  Other NFT projects on NEAR are already beginning to support NEP-4, so it's a good short-term choice.

### About Rust:

In the NEAR Pathway, the smart contract was written in AssemblyScript, which was then compiled to WebAssembly (WASM) to run in the blockchain.  However, NEAR smart contracts can also be written in [Rust], a C-like language for server applications that has become popular for its built-in safety checks that help avoid bugs.  Rust also features a robust testing infrastructure, copious on-line documentation, and a compiler that tries its best to help you fix any errors it finds.  

The NEAR team recommends using Rust for any smart contracts of a financial nature, and their reference implementation of NEP-4 NFTs is written in Rust.  So we will start with that reference implementation, and add some useful features. 

### Prerequisites:

If you have completed the NEAR Pathway, you should have already taken care of these prerequisites.  For this tutorial you must:
* Install node.js and npm, and set up your DataHub environment [(see Tutorial 1)](https://learn.figment.io/network-documentation/near/tutorials/1.-connecting-to-a-near-node-using-datahub)
* Create an account on the NEAR testnet [(see Tutorial 2)](https://learn.figment.io/network-documentation/near/tutorials/2.-creating-your-first-near-account-using-the-sdk)
* Install the NEAR CLI [(also in Tutorial 2)](https://learn.figment.io/network-documentation/near/tutorials/2.-creating-your-first-near-account-using-the-sdk)

## Installing The Toolchain

Before we can start working on the Rust contract, we need to install a few more tools.

### rustup

[rustup.rs] provides Rust installers for Unix and Windows platforms.  If you're using Unix, run the following command to install `rustup`, the Rust meta-installer:

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

This will also download and install `rustc`, the Rust compiler, and `cargo`, the Rust package manager. The installer will also add `$HOME/.cargo/bin` to your `PATH` environment variable.

(If you're using Windows, download and run the executable Rust installer from [rustup.rs].  The rest of this tutorial is written for a Unix environment, but the steps are essentially the same under Windows.)

Next we need to tell Rust to compile WebAssembly output (WASM) for the NEAR VM.  If your `rust` toolchain is missing the WASM components, the compiler will report an error similar to this one:

```
error[E0463]: can't find crate for `core`
  |
  = note: the `wasm32-unknown-unknown` target may not be installed
```

Run this command to add the WASM target to the Rust toolchain:

```
rustup target add wasm32-unknown-unknown
```

### yarn

If you haven't already, we need to install the `yarn` package manager.  The example code we're working with uses `yarn` as its build tool.  Run this  command to install `yarn`:

```
npm i -g yarn
```

If that all worked, you're ready to develop smart contracts in Rust.

### Cloning the NEAR NFT repo

In this tutorial we'll modify NEAR's NFT example code from the NEAR repository on Github. On Unix, run these commands in the `bash` shell to clone that repo and install its requirements:

```
git clone https://github.com/near-examples/NFT
cd NFT
yarn install
```

This repo contains NFT examples in both AssemblyScript and Rust, plus support files and documentation.  All the files we need for our smart contract live in the subdirectory `contracts/rust`

### Add Rust packages with Cargo

Rust includes an extensive ecosystem of support libraries (called [Crates]), and a package manager called Cargo to help you use them.  To create our NFT Flarns, we're going to add a few crates to this repo's Cargo manifest.  

Edit the file `contracts/rust/Cargo.toml`, and replace the entire contents with this:

```toml
[package]
name = "nep4-rs"
version = "0.1.0"
authors = ["NEAR Inc <hello@near.org>"]
edition = "2018"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0.60"
near-sdk = "2.0.0"
borsh = "0.7.1"
wee_alloc = "0.4.5"
rand = "=0.7.3"
rand_chacha = "=0.2.2"
rand_seeder = "=0.2.1"

[profile.release]
codegen-units = 1
# Tell `rustc` to optimize for small code size.
opt-level = "z"
lto = true
debug = false
panic = "abort"
# Opt into extra safety checks on arithmetic operations https://stackoverflow.com/a/64136471/249801
overflow-checks = true
```

We've only made one change to the original file: in the `[dependencies]` section we've added the `rand`, `rand_chacha` and `rand_seeder` crates.  Together they'll provide a random number generator that we will use to generate the unique DNA of our Flarns.

### Test the compiler

Every time the rust compiler runs it checks `Cargo.toml` for recent changes.  If it finds any, Cargo will automatically download those crates, build them, and cache them for future use.  

Lets run `cargo` manually now, to be sure we didn't make any typos in the manifest.  Type the following at the command line:

```
cargo verify-project --manifest-path contracts/rust/Cargo.toml
``` 

If there are no errors in the file, you should see this result in JSON format:

```
{"success":"true"}
```

Cargo is a powerful and important companion to Rust, and the `cargo` command has many options.  But this repo comes with pre-made build targets that will let us use `yarn` as our build tool, and `yarn` will use `cargo` under the hood. So that was the last time we'll run `cargo` directly. 
Instead, let's use `yarn` to make sure the contract in this repo is working before we modify it.  To build the example, enter this command in the shell:

```
yarn build:rs
```

Because this is our first build, Cargo and Rust will download and build all the dependencies.  If might take some time.  When it's finished, you should see something like this at the end of the output:

```
/Users/alice/NFT/contracts/rust
   Compiling nep4-rs v0.1.0 (/Users/alice/NFT/contracts/rust)
    Finished release [optimized] target(s) in 6.10s
✨  Done in 6.42s.
```

We can also run all of the included unit tests with this command:

```
yarn test:unit:rs
```

The unit test output is messy, but at the end you should see a summary of results.

```
test result: ok. 10 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

If you see `test result: ok`, all is well.

## Getting to know a Rust Contract

The Rust smart contract that we will modify is at `contracts/rust/src/lib.rs`.  Open that file in your editor.  We'll visit all the sections, and make a few important additions.

### Preamble

The first section is some boilerplate that imports useful features and configures the Rust compiler. 

```rust
#![deny(warnings)]

use borsh::{BorshDeserialize, BorshSerialize};
use near_sdk::collections::UnorderedMap;
use near_sdk::collections::UnorderedSet;
use near_sdk::{env, near_bindgen, AccountId};

#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;
```

### The NEP-4 Interface

Next comes a block labeled `pub trait NEP4`. This defines the API that we'll be able to use to interact with the smart contract.  A Trait in Rust is similar to an interface in Java or C++: it defines an API, but doesn't implement it.  The API here is the API for NFTs defined in the NEP-4 standard.  These 6 API methods allow a client to 

* Look up a token's owner, 
* Transfer a token between owners, 
* Assign or revoke token-transfer rights to another user

```rust
/// This trait provides the baseline of functions as described at:
/// https://github.com/nearprotocol/NEPs/blob/nep-4/specs/Standards/Tokens/NonFungibleToken.md
pub trait NEP4 {
    // Grant the access to the given `accountId` for the given `tokenId`.
    // Requirements:
    // * The caller of the function (`predecessor_id`) should have access to the token.
    fn grant_access(&mut self, escrow_account_id: AccountId);

    // Revoke the access to the given `accountId` for the given `tokenId`.
    // Requirements:
    // * The caller of the function (`predecessor_id`) should have access to the token.
    fn revoke_access(&mut self, escrow_account_id: AccountId);

    // Transfer the given `tokenId` to the given `accountId`. Account `accountId` becomes the new owner.
    // Requirements:
    // * The caller of the function (`predecessor_id`) should have access to the token.
    fn transfer_from(&mut self, owner_id: AccountId, new_owner_id: AccountId, token_id: TokenId);

    // Transfer the given `tokenId` to the given `accountId`. Account `accountId` becomes the new owner.
    // Requirements:
    // * The caller of the function (`predecessor_id`) should be the owner of the token. Callers who have
    // escrow access should use transfer_from.
    fn transfer(&mut self, new_owner_id: AccountId, token_id: TokenId);

    // Returns `true` or `false` based on caller of the function (`predecessor_id) having access to the token
    fn check_access(&self, account_id: AccountId) -> bool;

    // Get an individual owner by given `tokenId`.
    fn get_token_owner(&self, token_id: TokenId) -> String;
}

/// The token ID type is also defined in the NEP
pub type TokenId = u64;
pub type AccountIdHash = Vec<u8>;
```

### Blockchain storage

After the `trait` definition is the declaration of the contract's main data structure, `NonFungibleTokenBasic`.  Each of the methods in the NEP-4 trait will be implemented for that structure. It uses two `UnorderedMap`s to manage the ownership of our NFTs: `token_to_account` remembers the owner of each token, and `account_gives_access` remembers who has been granted transfer access by whom.  
`near_sdk::collections::UnorderedMap` is a custom hashmap implementation from the NEAR SDK, that's cheaper and more efficient to run on the chain than the standard hashmap provided by Rust, but is otherwise the same. `near_sdk::collections::UnorderedSet` is a similar structure, but simpler: it's just a set of keys, with no duplicates allowed.

```rust
// Begin implementation
#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize)]
pub struct NonFungibleTokenBasic {
    pub token_to_account: UnorderedMap<TokenId, AccountId>,
    pub account_gives_access: UnorderedMap<AccountIdHash, UnorderedSet<AccountIdHash>>, // Vec<u8> is sha256 of account, makes it safer and is how fungible token also works
    pub owner_id: AccountId,
}
```

### Customize metadata

That's technically everything our contract needs to manage the ownership of a set of Non-Fungible Token IDs.  But so far these token IDs don't refer to anything but themselves.  They have no metadata!  

Every type of NFT needs to define some metadata, to describe the individual interesting things being tracked: the name of a CryptoKitty, the title and author of an artwork, the date of a virtual ticketed event, or whatever else the NFT represents.  In fact, most NFTs are mostly metadata. So let's enhance this contract to store the metadata of each unique Flarn. 

Add the following lines of Rust code right beneath the comment that says `// Begin implementation`:

```rust
// Begin implementation
use near_sdk::serde::Serialize;
#[derive(Serialize, BorshDeserialize, BorshSerialize)]
pub struct Flarn {
    pub dna: u64,
}
```

This `Flarn` structure defines a minimal metadata for each CryptoFlarn: a single `dna` record that can be initialized with a unique random value. This is a very simple example, but consider that all of the variations we can see between individual CryptoKitties are derived from a similar block of random data, also called `dna`!

(The `#[derive]` attribute gives our Flarn the `BorschSerialize` trait, which lets the contract convert Flarns to a raw bytestream for storage and retrieval on NEAR.  Notice we didn't have to implement anything there, we just asked the compiler to figure it out for us.  [Derived traits] are a handy feature of Rust!  We're also using a related trait, `Serialize`, which lets the contract send Flarns over the network.) 

Next, we will update the definition of `NonFungibleTokenBasic` with one extra line of Rust. We'll add an `UnorderedMap` called `token_to_flarn` which will hold all the Flarn records in our smart contract.  

```rust
#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize)]
pub struct NonFungibleTokenBasic {
    pub token_to_account: UnorderedMap<TokenId, AccountId>,
    pub account_gives_access: UnorderedMap<AccountIdHash, UnorderedSet<AccountIdHash>>, // Vec<u8> is sha256 of account, makes it safer and is how fungible token also works
    pub owner_id: AccountId,
    pub token_to_flarn: UnorderedMap<TokenId, Flarn>,  // <-- ADD THIS LINE
}
```

Finally, add one line of Rust to the initializer for `NonFungibleTokenBasic`, so our flarns will be initialized when the smart contract is deployed to the blockchain :

```rust
#[near_bindgen]
impl NonFungibleTokenBasic {
    #[init]
    pub fn new(owner_id: AccountId) -> Self {
        assert!(env::is_valid_account_id(owner_id.as_bytes()), "Owner's account ID is invalid.");
        assert!(!env::state_exists(), "Already initialized");
        Self {
            token_to_account: UnorderedMap::new(b"token-belongs-to".to_vec()),
            account_gives_access: UnorderedMap::new(b"gives-access".to_vec()),
            owner_id,
            token_to_flarn: UnorderedMap::new(b"gives-flarn".to_vec()),  // <-- ADD THIS LINE 
        }
    }
}
```

### Browse the rest

 After the initializer for `NonFungibleTokenBasic` comes the implementaiton of the six API methods that make up the NEP4 trait. That's a big block of code. It won't all make sense if you're new to Rust, but parts of it may resemble other programming languages you know.  
 
 Fortunately, we won't need to change anything in this section.  If you do browse through it, you might notice:

* Methods in an `impl` block access their private instance of `NonFungibleTokenBasic` through an argument called `self`, similar to `this` in Javascript.
* Rust has an unusual `Some`/`None` construct for handling exceptions.
* Mutable variables are declared with the `mut` keyword; variables without `mut` are immutable by default.
* Only one or two lines near the end of each method actually modify our data. Most of this code deals with access control.
* The imported global `env` contains the ID of the NEAR user calling the contract, plus various other useful tidbits.

```rust
#[near_bindgen]
impl NEP4 for NonFungibleTokenBasic {
    fn grant_access(&mut self, escrow_account_id: AccountId) {
        let escrow_hash = env::sha256(escrow_account_id.as_bytes());
        let predecessor = env::predecessor_account_id();
        let predecessor_hash = env::sha256(predecessor.as_bytes());

        let mut access_set = match self.account_gives_access.get(&predecessor_hash) {
            Some(existing_set) => {
                existing_set
            },
            None => {
                UnorderedSet::new(b"new-access-set".to_vec())
            }
        };
        access_set.insert(&escrow_hash);
        self.account_gives_access.insert(&predecessor_hash, &access_set);
    }

    fn revoke_access(&mut self, escrow_account_id: AccountId) {
        let predecessor = env::predecessor_account_id();
        let predecessor_hash = env::sha256(predecessor.as_bytes());
        let mut existing_set = match self.account_gives_access.get(&predecessor_hash) {
            Some(existing_set) => existing_set,
            None => env::panic(b"Access does not exist.")
        };
        let escrow_hash = env::sha256(escrow_account_id.as_bytes());
        if existing_set.contains(&escrow_hash) {
            existing_set.remove(&escrow_hash);
            self.account_gives_access.insert(&predecessor_hash, &existing_set);
            env::log(b"Successfully removed access.")
        } else {
            env::panic(b"Did not find access for escrow ID.")
        }
    }

    fn transfer(&mut self, new_owner_id: AccountId, token_id: TokenId) {
        let token_owner_account_id = self.get_token_owner(token_id);
        let predecessor = env::predecessor_account_id();
        if predecessor != token_owner_account_id {
            env::panic(b"Attempt to call transfer on tokens belonging to another account.")
        }
        self.token_to_account.insert(&token_id, &new_owner_id);
    }

    fn transfer_from(&mut self, owner_id: AccountId, new_owner_id: AccountId, token_id: TokenId) {
        let token_owner_account_id = self.get_token_owner(token_id);
        if owner_id != token_owner_account_id {
            env::panic(b"Attempt to transfer a token from a different owner.")
        }

        if !self.check_access(token_owner_account_id) {
            env::panic(b"Attempt to transfer a token with no access.")
        }
        self.token_to_account.insert(&token_id, &new_owner_id);
    }

    fn check_access(&self, account_id: AccountId) -> bool {
        let account_hash = env::sha256(account_id.as_bytes());
        let predecessor = env::predecessor_account_id();
        if predecessor == account_id {
            return true;
        }
        match self.account_gives_access.get(&account_hash) {
            Some(access) => {
                let predecessor = env::predecessor_account_id();
                let predecessor_hash = env::sha256(predecessor.as_bytes());
                access.contains(&predecessor_hash)
            },
            None => false
        }
    }

    fn get_token_owner(&self, token_id: TokenId) -> String {
        match self.token_to_account.get(&token_id) {
            Some(owner_id) => owner_id,
            None => env::panic(b"No owner of the token ID specified")
        }
    }
}
```

### Make NFTs Mintable

After those NEP4 methods, there's a few more methods added to `NonFungibleTokenBasic` which aren't part of the NEP4 standard.  For instance, because NEP4 doesn't say anything about how NFTs are created, this example contains a `mint_token()` method that can create them on demand.  `mint_token()` takes two arguments: a token ID and an owner ID.  

Let's modify `mint_token()` to mint our flarns.  We'll import the random number generator that we added to the manifest earlier, and use it to initialize each new flarn's DNA with a random 64-bit value.

Replace the entire block underneath `/// Methods not in the strict scope of the NFT spec` with this code:

```rust
/// Methods not in the strict scope of the NFT spec (NEP4)
#[near_bindgen]
impl NonFungibleTokenBasic {
    /// Creates a token for owner_id, doesn't use autoincrement, fails if id is taken
    pub fn mint_token(&mut self, owner_id: String, token_id: TokenId) {
        // make sure that only the owner can call this funtion
        self.only_owner();
        // Since Map doesn't have `contains` we use match
        let token_check = self.token_to_account.get(&token_id);
        if token_check.is_some() {
            env::panic(b"Token ID already exists.")
        }
        // No token with that ID exists, mint and add token to data structures
        self.token_to_account.insert(&token_id, &owner_id);

        // Generate random Flarn DNA:
        use rand::prelude::*;
        use rand_chacha::ChaCha8Rng;
        use rand_seeder::{Seeder};
        let mut rng: ChaCha8Rng = Seeder::from(env::random_seed()).make_rng();
        let new_flarn = Flarn {dna: rng.gen()};
        self.token_to_flarn.insert(&token_id, &new_flarn);
    }

    /// helper function determining contract ownership
    fn only_owner(&mut self) {
        assert_eq!(env::predecessor_account_id(), self.owner_id, "Only contract owner can call this method.");
    }

    /// Added method: get the metadata for a token
    pub fn get_token_meta(&self, token_id: TokenId) -> Flarn {
        match self.token_to_flarn.get(&token_id) {
            Some(flarn) => flarn,
            None => env::panic(b"Missing metadata.")
        }
    }
}
```

We've changed two things here:
* We added code in `mint_token()` to create a new Flarn metadata record, fill its dna with random data, and save that to our metadata hash
* We've added a new contract method `token_to_meta()` to return a token's metadata when we pass its ID

That's the end of the smart contract. The rest of the code in this file is unit tests.

### Add a test

In Rust, unit tests usually live in the same file as the code being tested, wrapped in a `mod tests` block, and decorated with `#[test]` attributes that show the compiler where to find each test.  It's a good practice to write at least one test for every new feature we add.  Let's add a test for our new `token_to_meta()` method.

Scroll to the bottom of `lib.rs`.  The very last line in the file has just one closing bracket.  Replace that entire line (including the final  bracket) with this block of code (including the final bracket):

```rust
    #[test]
    fn token_to_meta() {
        // Make an instance of the contract, and set up a test context
        let context = get_context(robert(), 0);
        testing_env!(context);
        let mut contract = NonFungibleTokenBasic::new(robert());
        // Mint a token
        contract.mint_token(mike(), 19u64);
        // Get the token's metadata
        let metadata = contract.get_token_meta(19u64);
        // check that the DNA contains a value:
        assert!(metadata.dna != 0, "DNA not set.");
    }
} 
```

### Check our work

Now that we've added some code, let's try building and testing again, and see if we broke anything.  As before, type this command to build the project:

```
yarn build:rs
```

If you have made any errors, the compiler should give you a detailed explanation.  Since you've copied and pasted everything from this tutorial, it's probably just typos.  But if you can't fix it with the compiler's help, replace the entire lib.rs file with [this version](https://github.com/figment-networks/tutorials/tree/main/near/6_NFT/contracts/rust/src/lib.rs), which contains all the changes we've made so far. 

Once that's building correctly, try running the unit tests:

```
yarn test:unit:rs
```

You'll see that there are now 11 unit tests running, and they should all pass.

(After running the unit tests Yarn will try to run documentation tests -- but there aren't any here. That's why you'll see some output about `running 0 tests`.  You can ignore that part.)

```
(output)
```

## Deploying and using the contract

We can use the NEAR CLI to deploy this contract, and to test that it's working.  If you configured your environment in Tutorials 1 and 2, the CLI will connect to DataHub's high-availability testnet.  If you don't have DataHub access you can still run the CLI with its defaults, though the default testnet node may be slower to respond.  Run this command to deploy the contract you just built:

```
near dev-deploy out/nep4_rs.wasm 
```

The output will show details of the deployment transaction, and the ID of the test NEAR account that the CLI auto-generated for you.  It should look something like this:

```
Starting deployment. Account id: dev-1607722059840-7354752, node: https://near-testnet--rpc.datahub.figment.io/apikey/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx, helper: https://helper.testnet.near.org, file: out/nep4_rs.wasm
Transaction Id DanbgVsY3VCsQMh2zQvAGKuYWe1WFBuwsJo2oMti5QkC
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.testnet.near.org/transactions/DanbgVsY3VCsQMh2zQvAGKuYWe1WFBuwsJo2oMti5QkC
Done deploying to dev-1607722059840-7354752
```

The provided link will give you complete details about the deployment in the NEAR Explorer.

[ screenshot ](https://github.com/figment-networks/datahub-learn/tree/master/.gitbook/assets/screenshot-near-rust-tut.png)

In this step, the CLI created a new user account on the testnet and deployed the contract in that account.  Make a note of that new Account ID, which looks something like `dev-nnnnnnnnn-nnnnn`, where the `n`s are replaced by digits.

### Initialize the contract

Our NFT smart contract is now deployed on NEAR!  Let's use the CLI to test this interface. 

### contract arguments
First, we we need to call our contract's `new()` method, to initialize the blockchain data store.  If we call any other method before that, we'll get an error.  The 'new' method takes one argument, `owner_id`, the accountID of the user who will be allowed to mint new Flarns from this contract.

### CLI arguments
To call that `new()` method from the CLI, run this command, replacing `CALLER-ID`, `RECEIVER-ID` and `OWNER-ID`  with the test account ID you got from the previous step.  (Here, `CALLER-ID` is the account the CLI will use to make the call, `RECEIVER-ID` is the account where the contract is deployed, and `OWNER-ID` is the account that the contract will authorize to mint Flarns. For this test, we'll use the same account in all three roles.)

```
near call --accountId CALLER-ID RECEIVER-ID new '{"owner_id": "OWNER-ID"}'
```

The output should return a Transaction ID and a link to the NEAR Explorer:

```
Scheduling a call: dev-1607722059840-7354752.new({"owner_id": "dev-1607722059840-7354752"})
Transaction Id 9PZZFWJUco7f33vJEjTbcjzsGhiigtYiSSyNh5FmJdNb
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.testnet.near.org/transactions/9PZZFWJUco7f33vJEjTbcjzsGhiigtYiSSyNh5FmJdNb
```

## Mint an NFT!

Now we'll make a call to `mint_token()`.  We'll need a block of JSON containing the two arguments to that method: an ID for the token, which can be any integer, and an account ID of the token's first owner.  We'll use the same test account ID as before, and give `1234` as a token ID.  

Run this at your command line, again substituting `CALLER-ID` and `RECEIVER-ID`:

```
near call --accountId CALLER-ID RECEIVER-ID mint_token '{ "token_id":1234, "owner_id": "CALLER-ID"}'
```

The output will look pretty similar to the output of the `new()` method.  Neither of those methods return any data.  But the new transaction ID and the explorer link should confirm for us that the token was minted.

Still, we can be even more sure.  Let's fetch the metadata for our newly minted token and have a look at it.  Run this command, again replacing `ACCOUNT-ID` and `CONTRACT-ID`:

```
near view --accountId ACCOUNT-ID CONTRACT-ID get_token_meta '{"token_id":1234}'
```

The output is similar, but the last line contains the return value in JSON:

```
{ dna: 9932525801968679000 }
```

The actual number you see will be different from this, if the random number generator is good at generating random numbers.  But here is our NFT metadata, stored in the NEAR blockchain.

## Conclusion

You now have deployed an NFT smart contract on the NEAR testnet, and have minted one CryptoFlarn NFT.  From here you could use the CLI or the NEAR Javascript SDK to transfer ownership of that token, or to make more tokens.  NFT marketplaces are already under development on NEAR; when they support NEP-4, you'll be able to trade these tokens there.  If your next NFT project needs more complex metadata, you've seen how that can be added.

The complete code for this tutorial can be found on [Github](https://github.com/figment-networks/tutorials/tree/main/near/6_NFT).


[rustup.rs]:https://rustup.rs/
[Non-Fungible Tokens]:https://en.wikipedia.org/wiki/Non-fungible_token
[Cryptokitties]:https://www.cryptokitties.co/
[SuperRare]:https://superrare.co/
[ERC721]:https://eips.ethereum.org/EIPS/eip-721
[NEP-4]:https://github.com/nearprotocol/NEPs/pull/4
[NEAR EVM]:https://near.org/blog/running-ethereum-applications-on-near/
[Derived traits]:https://doc.rust-lang.org/rust-by-example/trait/derive.html
[Rust]:https://www.rust-lang.org/
[Crates]:https://crates.io/
