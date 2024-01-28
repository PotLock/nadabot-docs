---
description: For application looks for a sybil resistance Solution
---

# 📱 For Applications

If you want to integrate us this usually means you are building an application on NEAR and either

* **Front End Gating:** check a user logged is a human to display different views
* **NEAR Contracts**: using for a cross contract call to our registry to enforce on the contract level

## Front End Gating

Check out the NEAR Docs to get a better sense of how to integrate in the front end. If you are using a javascript front end library you will most likely be integrating with our contract via NEAR API JS



```bash
npm install near-api-js near-wallet-selector
```

{% embed url="https://docs.near.org/tools/near-api-js/quick-reference" %}

In your javascript app

```javascript
import { connect, keyStores, WalletConnection } from 'near-api-js';
import { setupWalletSelector } from 'near-wallet-selector';

async function initNear() {
    const nearConfig = {
        networkId: "mainnet", // or "testnet"
        keyStore: new keyStores.BrowserLocalStorageKeyStore(),
        nodeUrl: "https://rpc.mainnet.near.org", // or "https://rpc.testnet.near.org" for testnet
        walletUrl: "https://app.mynearwallet.com", // or "https://wallet.testnet.near.org" for testnet
    };

    // Initialize connection to the NEAR protocol
    const near = await connect(nearConfig);

    // Initialize Wallet Selector
    const walletSelector = await setupWalletSelector({ 
        network: nearConfig.networkId,
        modules: [/* ... */],
    });

    // Initialize Wallet Connection
    const wallet = new WalletConnection(near, null);

    return { wallet, walletSelector, near };
}

async function checkIsHuman(accountId) {
    const { wallet } = await initNear();

    if (!wallet.isSignedIn()) {
        // Show login popup modal here
        // Example: showModal("Please sign in to continue");
        return;
    }

    const contract = new wallet.account().contract({
        viewMethods: ['isHuman'],
        changeMethods: [],
    });

    try {
        const isHuman = await contract.isHuman({ account_id: accountId });
        let grantAccess = isHuman;
        console.log("Grant Access: ", grantAccess);

        // Additional logic based on grantAccess
    } catch (error) {
        console.error("Error checking if human: ", error);
    }
}

// Replace with the actual account ID to check
const accountIdToCheck = "example-account.near";
checkIsHuman(accountIdToCheck);

```

This code does the following:

1. Initializes a connection to the NEAR blockchain using the `near-api-js` library.
2. Sets up the NEAR Wallet Selector, which allows users to log in using various NEAR wallets.
3. Checks if the user is logged in; if not, shows a popup modal prompting them to sign in.
4. Once logged in, it calls the `isHuman` view method on the `sybil.potlock.near` contract, passing the specified `accountId`.
5. The result (true/false) is saved in the `grantAccess` variable and can be used for further logic.

Make sure to handle the UI elements (like the popup modal) according to your application's frontend framework or library. The above code assumes a basic JavaScript setup and will need to be adapted to fit your specific implementation details.

## NEAR Contract

NEAR Protocl is a layer 1 blockchain with Rust contracts that compile to Wasm.&#x20;

**Create Your First Rust Contract**: After installing Rust, you can create your first Rust contract. Here are the steps:

* Download and install [Rust](https://doc.rust-lang.org/book/ch01-01-installation.html).
* Create a new Rust project using the [quickstart guide](https://app.hzn.xyz/2.develop/quickstart.md).
* Read the NEAR docs on [how to write smart contracts](https://app.hzn.xyz/2.develop/contracts/anatomy.md).

**Resources**: For more information and examples, you can refer to the following resources:

* Documentation: https://docs.near.org/develop/contracts/anatomy&#x20;
* Examples: https://docs.near.org/tutorials/welcome
* Github: https://github.com/near/near-sdk-rs

### Cross Contract Calls

{% embed url="https://github.com/near/docs/blob/master/docs/sdk/rust/cross-contract/callbacks.md" %}

Here is an example with how nada.bot does it with I-Am Human contract [https://github.com/PotLock/core/blob/71f35051f81c333c85fafc42d073173191ad14ca/contracts/sybil/src/human.rs#L43](https://github.com/PotLock/core/blob/71f35051f81c333c85fafc42d073173191ad14ca/contracts/sybil/src/human.rs#L43)



To make a cross-contract call in Rust for a NEAR smart contract, you would typically use the `promise_then` method to schedule the execution of a callback after the initial cross-contract call completes. Here's a basic example of how you might structure this in your Rust smart contract:

```
```



```rust
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::{env, near_bindgen, Promise, AccountId};
use near_sdk::json_types::U128;

#[near_bindgen]
#[derive(Default, BorshDeserialize, BorshSerialize)]
pub struct MyContract {
    // Your contract's state and fields go here
}

#[near_bindgen]
impl MyContract {
    pub fn call_is_human(&self, account_id: AccountId) {
        let sybil_contract: AccountId = "v1.nadabot.near".parse().unwrap();

        // Call `isHuman` method on the sybil_contract
        Promise::new(sybil_contract)
            .function_call(
                "isHuman".to_string(),
                // Here you would serialize the parameters for the `isHuman` method
                // For this example, assume that it takes an account_id in JSON format
                serde_json::to_vec(&account_id).unwrap(),
                0, // attached deposit
                env::prepaid_gas() / 2 // use half of the remaining gas
            )
            .then(
                // Schedule the execution of the callback method
                env::current_account_id(),
                "on_is_human_callback".to_string(),
                // You can pass any additional data needed for the callback here
                // In this case, we're not passing any extra data
                vec![],
                0, // attached deposit for the callback
                env::prepaid_gas() / 2 // use the remaining half of the gas
            );
    }

    // Callback function
    pub fn on_is_human_callback(&mut self) {
        // Here you would handle the result from the `isHuman` call
        // Assuming the result is a boolean
        let is_human: bool = match env::promise_result(0) {
            near_sdk::PromiseResult::Successful(result) => {
                // Deserialize the result to a bool
                serde_json::from_slice::<bool>(&result).unwrap_or(false)
            }
            _ => false,
        };

        // Store or use `is_human` as needed
        let is_valid = is_human;
        // Further logic depending on `is_valid`
    }
}

// Boilerplate code for tests and initialization goes here

```

###

## Ideas to Integrate

* Airdropping and amplified rewards for loyalty programs
* 1 person 1 vote for governance on governance contract
