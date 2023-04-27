# Q2_Prereqs

# Lesson One: Enrollment dApp

In this lesson, we are going to: 
1. Learn how to use `@solana/web3.js` to create a new keypair
2. Use our Public Key to airdrop ourselves some Solana devnet tokens
3. Make Solana transfers on devnet
4. Empty your devnet wallet into your WBA wallet


Prerequisites:
1. Have NodeJS installed
2. Have yarn installed
3. Have a fresh folder created to follow this tutorial and all future tutorials

Let's get into it!

## 1. Create a new Keypair

To get started, we're going to create a keygen script and an airdrop script for our account.

#### 1.1 Setting up
Start by opening up your Terminal. We're going to use `yarn` to create a new Typescript project.

```sh
mkdir airdrop && cd airdrop
yarn init -y
```

Now that we have our new project initialised, we're going to go ahead and add `typescript`, `bs58` and `@solana/web3.js`, along with generating a `tsconfig.js` configuration file.

```sh
yarn add @types/node typescript @solana/web3.js bs58
yarn add -D ts-node
touch keygen.ts
touch airdrop.ts
touch transfer.ts
yarn tsc --init --rootDir ./ --outDir ./dist --esModuleInterop --lib ES2019 --module commonjs --resolveJsonModule true --noImplicitAny true
```

Finally, we're going to create some scripts in our `package.json` file to let us run the three scripts we're going to build today:

```js
{
  "name": "airdrop",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "keygen": "ts-node ./keygen.ts",
    "airdrop": "ts-node ./airdrop.ts",
    "transfer": "ts-node ./transfer.ts",

  },
  "dependencies": {
    "@solana/web3.js": "^1.75.0",
    "@types/node": "^18.15.11",
    "typescript": "^5.0.4"
  },
  "devDependencies": {
    "ts-node": "^10.9.1"
  }
}
```

Alright, we're ready to start getting into the code!

#### 1.2 Generating a Keypair
Open up `./keygen.ts`. We're going to generate ourselves a new keypair.

We'll start by importing `Keypair` from `@solana/web3.js`

```ts
import { Keypair } from "@solana/web3.js";
```

Now we're going to create a new Keypair, like so:

```ts
//Generate a new keypair
let kp = Keypair.generate()
console.log(`You've generated a new Solana wallet: ${kp.publicKey.toBase58()}

To save your wallet, copy and paste the following into a JSON file:

[${kp.secretKey}]`)
```

Now we can run the following script in our terminal to generate a new keypair!

```sh
yarn keygen
```

This will generate a new Keypair, outputting its Address and Private Key like so:
```
You've generated a new Solana wallet: 2sNvwMf15WPp94kywgvfn3KBNPNZhr5mWrDHmgjkjMhN

To save your wallet, copy and paste your private key into a JSON file:

[34,46,55,124,141,190,24,204,134,91,70,184,161,181,44,122,15,172,63,62,153,150,99,255,202,89,105,77,41,89,253,130,27,195,134,14,66,75,242,7,132,234,160,203,109,195,116,251,144,44,28,56,231,114,50,131,185,168,138,61,35,98,78,53]
```

If we want to save this wallet locally. To do this, we're going run the following command:

```sh
touch dev-wallet.json
```
This creates the file `dev-wallet.json` in our `./airdrop` root directory. Now we just need to paste the private key from above into this file, like so:

```json
[34,46,55,124,141,190,24,204,134,91,70,184,161,181,44,122,15,172,63,62,153,150,99,255,202,89,105,77,41,89,253,130,27,195,134,14,66,75,242,7,132,234,160,203,109,195,116,251,144,44,28,56,231,114,50,131,185,168,138,61,35,98,78,53]
```

Congrats, you've created a new Keypair and saved your wallet. Let's go claim some tokens!

## 2. Claim Token Airdrop
Now that we have our wallet created, we're going to import it into another script

This time, we're going to import `Keypair`, but we're also going to import `Connection` to let us establish a connection to the Solana devnet, and `LAMPORTS_PER_SOL` which lets us conveniently send ourselves amounts denominated in SOL rather than the individual lamports units.

```ts
import { Connection, Keypair, LAMPORTS_PER_SOL } from "@solana/web3.js"
```

We're also going to import our wallet and recreate the `Keypair` object using its private key:

```ts
import wallet from "./dev-wallet.json"

// We're going to import our keypair from the wallet file
const keypair = Keypair.fromSecretKey(new Uint8Array(wallet));
```

Now we're going to establish a connection to the Solana devnet:
```ts
//Create a Solana devnet connection to devnet SOL tokens
const connection = new Connection("https://api.devnet.solana.com");
```

Finally, we're going to claim 2 devnet SOL tokens:
```ts
(async () => {
    try {
        // We're going to claim 2 devnet SOL tokens
        const txhash = await connection.requestAirdrop(keypair.publicKey, 2 * LAMPORTS_PER_SOL);
        console.log(`Success! Check out your TX here: 
        https://explorer.solana.com/tx/${txhash}?cluster=devnet`);
    } catch(e) {
        console.error(`Oops, something went wrong: ${e}`)
    }
})();
```

Here is an example of the output of a successful airdrop:

```
Success! Check out your TX here:
https://explorer.solana.com/tx/459QHLHJBtkHgV3BkzGKo4CDSWzNr8HboJhiQhpx2dj8xPVqx4BtUPCDWYbbTm426mwqmdYBhEodUQZULcpvzd5z?cluster=devnet
```

## 3. Transfer tokens to your WBA Address
Now we have some devnet SOL to play with, it's time to create our first native Solana token transfer. When you first signed up for the course, you gave WBA a Solana address for certification. We're going to be sending some devnet SOL to this address so we can use it going forward.

We're gong to open up `transfer.ts` and import the following items from `@solana/web3.js`:
```ts
import { Transaction, SystemProgram, Connection, Keypair, LAMPORTS_PER_SOL, sendAndConfirmTransaction, PublicKey } from "@solana/web3.js"
```

We will also import our dev wallet as we did last time:
```ts
import wallet from "./dev-wallet.json"

// Import our dev wallet keypair from the wallet file
const from = Keypair.fromSecretKey(new Uint8Array(wallet));

// Define our WBA public key
const to = new PublicKey("GLtaTaYiTQrgz411iPJD79rsoee59HhEy18rtRdrhEUJ");
```

And create a devnet connection:
 
```ts
//Create a Solana devnet connection
const connection = new Connection("https://api.devnet.solana.com");
```

Now we're going to create a transaction using `@solana/web3.js` to transfer 0.1 SOL from our dev wallet to our WBA wallet address on the Solana devnet. Here's how we do that:
```ts
(async () => {
    try {
        const transaction = new Transaction().add(
            SystemProgram.transfer({
                fromPubkey: from.publicKey,
                toPubkey:  to,
                lamports: LAMPORTS_PER_SOL/100,
            })
        );
        transaction.recentBlockhash = (await connection.getLatestBlockhash('confirmed')).blockhash;
        transaction.feePayer = from.publicKey;
        
        // Sign transaction, broadcast, and confirm
        const signature = await sendAndConfirmTransaction(
            connection,
            transaction,
            [from]
        );
        console.log(`Success! Check out your TX here: 
        https://explorer.solana.com/tx/${signature}?cluster=devnet`);
    } catch(e) {
        console.error(`Oops, something went wrong: ${e}`)
    }
})();
```

## 4. Empty devnet wallet into WBA wallet

Okay, now that we're done with our devnet wallet, let's also go ahead and send all of our remaining lamports to our WBA dev wallet. It is typically good practice to clean up accounts where we can as it allows us to reclaim resources that aren't being used which have actual monetary value on mainnet.

To send all of the remaining lamports out of our dev wallet to our WBA wallet, we're going to need to add in a few more lines of code to the above examples so we can:
1. Get the exact balance of the account
2. Calculate the fee of sending the transaction
3. Calculate the exact number of lamports we can send whilst satisfying the fee rate

```ts
(async () => {
    try {
        // Get balance of dev wallet
        const balance = await connection.getBalance(from.publicKey)

        // Create a test transaction to calculate fees
        const transaction = new Transaction().add(
            SystemProgram.transfer({
                fromPubkey: from.publicKey,
                toPubkey: to,
                lamports: balance,
            })
        );
        transaction.recentBlockhash = (await connection.getLatestBlockhash('confirmed')).blockhash;
        transaction.feePayer = from.publicKey;

        // Calculate exact fee rate to transfer entire SOL amount out of account minus fees
        const fee = (await connection.getFeeForMessage(transaction.compileMessage(), 'confirmed')).value || 0;

        // Remove our transfer instruction to replace it
        transaction.instructions.pop();

        // Now add the instruction back with correct amount of lamports
        transaction.add(
            SystemProgram.transfer({
                fromPubkey: from.publicKey,
                toPubkey: to,
                lamports: balance - fee,
            })
        );

        // Sign transaction, broadcast, and confirm
        const signature = await sendAndConfirmTransaction(
            connection,
            transaction,
            [from]
        );
        console.log(`Success! Check out your TX here: 
        https://explorer.solana.com/tx/${signature}?cluster=devnet`)
    } catch(e) {
        console.error(`Oops, something went wrong: ${e}`)
    }
})();
```

As you can see, we crated a mock version of the transaction to perform a fee calculation before removing and readding the transfer instruction, signing and sending it. You can see from the outputted transaction signature on the block explorer here that the entire value was sent to the exact lamport:
```
Check out your TX here (This is the TX Hash you will submit in the submission form):
https://explorer.solana.com/tx/4dy53oKUeh7QXr15wpKex6yXfz4xD2hMtJGdqgzvNnYyDNBZXtcgKZ7NBvCj7PCYU1ELfPZz3HEk6TzT4VQmNoS5?cluster=devnet
```

