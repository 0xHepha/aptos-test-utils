# Test Utils (tu::)

This is a Move module for the Aptos Blokchain. The main objective is to make tests easier to write while making them as clean as posible, it is mostly oriented towards coin and collection management but other functions aswell like time or randomness initialization.

![code example](https://arweave.net/grWA0UizUm3Myx4aiKnxv0U3DKNrQioSLO0Xh-Qcr1c)

> [!WARNING]
> Don't use the system or ideas of this module outside of testing functions! It is slow, expensive and dangerous.This module is focused on making it comfortable for devs when writing test, not for production.

> [!IMPORTANT]  
> This module was build using MOVE2, while using it, remember to add `--move-2` flag


## Supported features

-   V1 & V2 Collections creation and management
-   V1 & V2 Tokens creation and management
-   10 different types of Coin minting
-   APT mint and initialization for addresses
-   Timestamp initialization
-   Randomness initialization

## Posible future updates ideas

-   Simple management of time functions: `time.start()`, `time.addSeconds()`, `time.addDays()`, `time.stringFormat()`,etc
-   Easy management of randomness for testing: `lucky.setOucome(1)`,etc
-   Posibility to add external `Structs` to `Item` system to use its features.

## How to implement

1. Add `test_utils.move` inside your `sources/` folder. Either download the file or create a new one and copy-paste the whole code.
2. Code deploys the module as `module deployer::test_utils{}`, you have two choices:
    1. Add a deployer address under `[addresses]` in your `Move.toml` file
    2. Change code at top to `module [address_name]::test_utils{}`

## How to use

### 1. import the module.

```Rust
#[test_only]
use deployer::tu::{Self, Coin1, Coin2};
```

### 2. Add `aptos_framework` to it:

1. Add `aptos_framework = "0x1"` under `[addresses]` in your `Move.toml`
2. Add the framework to all test functions where you use it, example:

```Rust
#[test(aptos_framework = @0x1,deployer = @deployer)]
fun test__my_test(aptos_framework: &signer, deployer: &signer){}
```

### 3. Intialize the module at

At the begining of each function call: `tu::init(aptos_framework,deployer);`

### Full intialization example

```Rust
#[test_only]
use deployer::tu::{Self, Coin1, Coin2};

#[test(aptos_framework = @0x1,deployer = @deployer)]
fun test__my_test(aptos_framework: &signer, deployer: &signer){
    tu::init(aptos_framework, deployer);

    /// Rest of code...
}
```

## Examples

#### Coin functionalities use example:

```Rust
#[test_only]
use deployer::tu::{Self, Coin1, Coin2};

#[test(aptos_framework = @0x1,deployer = @deployer, user1 = @user1, user2 = @user2)]
fun test__pool(aptos_framework: &signer, deployer: &signer, user1: &signer, user2: &signer){
    tu::init(aptos_framework, deployer);

    // Intialize all accounts for test and give them some APT
    tu::get_apt(deployer, 10);
    tu::get_apt(user1, 2);
    tu::get_apt(user2, 2);

    // Rest of code...
}
```

#### Collection features example:

```Rust
#[test_only]
use deployer::tu::{Self, Coin1, Coin2};

#[test(aptos_framework = @0x1,deployer = @deployer, user1 = @user1, user2 = @user2)]
fun test__marketplace(aptos_framework: &signer, deployer: &signer, user1: &signer, user2: &signer){
    tu::init(aptos_framework, deployer);

    // Intialize accounts and give some apt
    tu::get_apt(user1, 2);
    tu::get_apt(user2, 2);

    // Create a v1 and v2 collection
    let collection_1 = tu::create_collection_v1();
    let collection_2 = tu::create_collection_v2();

    // Mint 3 tokens of each collection to user1
    collection_1.mint(user1, 3);
    collection_2.mint(user1, 3);

    // Mint 2 tokens to users2
    collection_1.mint(user2, 2);

    // get user1 address
    let user1_address = signer::address_of(user1);

    // Get all tokens of user1
    let user_tokens = tu::tokens_of(user1_address);

    // Get first token of collection 1
    let token1 = collection_1.token_at(0);

    // Check if user1 is the owner
    if(token1.is_owner(user1_address)){
        // get token info
        let token_name = token1.name();
        let token_collection_name = token.collection_name();

        // Send token1 to user2
        token1.send(user1, user2);
    }
}
```

## Documentation

The main objective of this module is to be able to use somthing similar to object functions. All of the `self` argument functions use the `Item` structure.

> [!IMPORTANT]  
> Read bellow on how functions are noted and how to use them:
> **Normal functions** will be documented as `tu::function_name()` in this doc

**ALL of the functions** that don't start with `tu::` in this doc are special functions linked to objects and they must be called using **dot syntax**. Those functions are declared as `get_item(self: &Item, index: u64)` but in this doc we will refere to them as `get_item(index: u64)`

Example:

```Rust
// Normal functions calls that return an Item
let collection_1 = tu::create_collection_v1();
let collection_2 = tu::create_collection_v2();

// name() is a special function, so it needs to be called from a object
// In this case we use on V1 and V2 collections but it supports v1/v2 tokens aswell
let collection_name_1 = collection_1.name();
let collection_name_2 = collection_2.name()
```

### Coins

There are 10 Test Coins created already with 1M supply on each. Supported `<CoinType>` ranges from `<Coin1>` to `<Coin10>`. Both coin related functions deposit the coins/apt directlly to the signers address.

-   `get_coins<CoinType>(to: &signer, amount: u64)`
-   `get_apt(to: &signer, amount: u64)` This function also initializes the accounts before transfering APT to them, its a great function to call at the begining of the test function for all the accounts.

### Collections

Take into account that v1 and v2 collections don't afect each others index, there are 2 separate lists to store them, meaning that if you create three v1 collections and then one v2 collection, the index of the v2 collection is `0` since its the frist v2 collection, the call to get it would be `tu::get_collection_at(0,2)`

#### Collection V1

-   `tu::get_collection_at(index: u64, type: u64):Item(Collection)` use `1` for type parameter
-   `tu::create_collection_v1():Item(CollectionV1)`
-   `mint(minter: &signer,amount: u64)` Mints a v1 Tokens
-   `type():u64` indicates the type of collection v1 or v2
-   `index():64` index of this collection inside the list that stores v1 collections
-   `creator():address`
-   `name():String`
-   `supply():u64`
-   `tokens():vector<Item(TokenV1)>`
-   `token_at(index: u64):Item(CollectionV1)`

#### Token V1

-   `type():u64` indicates the type of Token v1 or v2
-   `index():u64` index of this token inside its collection
-   `creator():address`
-   `name():String`
-   `collection_name():String`
-   `collection_index():u64`
-   `version():u64` property version of this token, check `0x3::token` for extra info
-   `is_owner(user: address):bool`
-   `id():TokenId` The structure that it returns is declared in `0x3::token`
-   `transfer(from: &signer, to: &signer)`
-   `tu::tokens_of(user:address):vector<Item(Token)>` Returns all the v1 and v2 tokens of address
-   `tu::tokens_amount(user:address):u64`

#### Collection V2

-   `tu::get_collection_at(index: u64, type: u64):Item(Collection)` use `2` for type parameter
-   `tu::create_collection_v2():Item(CollectionV2)`
-   `mint(minter: &signer, amount: u64)` Mints v2 Tokens
-   `type():u64` indicates the type of collection v1 or v2
-   `index():u64` index of this collection inside the list that stores v2 collections
-   `creator():address`
-   `supply():u64`
-   `addr():address` Address of the collection object
-   `tokens():vector<Item(TokenV2)>`
-   `token_at(index: u64):Item(TokenV2)`

#### Token V2

-   `type():u64` indicates the type of Token v1 or v2
-   `index():u64` index of this token inside its collection
-   `addr():address` Address of the token object
-   `collection_address():address`
-   `collection_index():u64`
-   `is_owner(user: address):bool`
-   `object():<aptos_token::AptosToken>`
-   `transfer(from: &signer, to: &signer)`
-   `tu::tokens_of(user:address):vector<Item(Token)>` Returns all the v1 and v2 tokens of address
-   `tu::tokens_amount(user:address):u64`

## Contributing

I build this module since I needed something that let me focus on testing my new code insetead of having most of my time managing tokens or collections, thus the initial functionalities might be a bit skewed towards my necessities, thats why anyone who wants to improve is welcomed.

I you have an idea on how to improve it or just something that you find would be usefull for you if added, please contact me and share your ideas:

-   Discord: 0xhepha
-   Email: 0xhepha@gmail.com
