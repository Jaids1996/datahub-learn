With some protocols, different networks (testnet, mainnet, etc) have different token names. For example with Polkadot, the mainnet token is **DOT** and the testnet token is **WND**. In the Solana world, the token is always called **SOL**, no matter what network (or **cluster**) you are on. Don't get too excited: the tokens you get for free on the devnet cannot be used on Solana's mainnet. Nice try though 😉

----------------------------------

# Airdropping

To fund an account, we will do what is called an **airdrop** - some tokens will magically fall from the sky into our wallets! The cluster will provide us with some **SOL** so that we can test making transfers as well as view the transaction details on a block explorer.

{% hint style="info" %}
1 **SOL** is equal to 1,000,000,000 **lamports**. The name of **lamports** is in honour of Solana's biggest technical influence, [Leslie Lamport](https://en.wikipedia.org/wiki/Leslie_Lamport).
{% endhint %}

----------------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/solana/fund.ts`, implement the `fund` function. Convert the text input to an address and use `requestAirdrop` to get 1 **SOL**. You must replace the instances of `undefined` with working code to accomplish this.
{% endhint %}

**Take a few minutes to figure this out.**

```typescript
//..
  try {
    const {network, address} = req.body;
    const url = getNodeURL(network);
    const connection = new Connection(url, "confirmed");
    const publicKey = undefined;
    const hash = undefined;
    await undefined;
    res.status(200).json(hash);
  }
//..
```

**Need some help?** Here are a few hints.
* [Create a publicKey from a string](https://solana-labs.github.io/solana-web3.js/classes/PublicKey.html#constructor)  
* [`requestAirdrop` documentation](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#requestairdrop)


{% hint style="info" %}
You can [**join us on Discord**](https://figment.io/devchat), if you have questions or want help completing the tutorial.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# Solution

```typescript
// solution
//..
  try {
    const {network, address} = req.body;
    const url = getNodeURL(network);
    const connection = new Connection(url, 'confirmed');
    const publicKey = new PublicKey(address);
    const hash = await connection.requestAirdrop(publicKey, LAMPORTS_PER_SOL);
    await connection.confirmTransaction(hash);
    res.status(200).json(hash);
  }
//..
```

**What happened in the code above?**

* We created a `PublicKey` from the string formatted address.
* We pass this to `requestAirdrop`, together with a constant which represents one `SOL`
* We can then verify than the transaction is confirmed by passing the transaction hash to the `confirmTransaction` method.
* Finally, we return the hash of the transaction to the client side in JSON format.

----------------------------------

# Make sure it works

Once you have the code above saved:
* Copy and paste the generated address into the text input.   
* Click on **Fund this Address** 

Let the magic happen: You're now 1 SOL richer on devnet!

![](../assets/solana/solana-fund.gif)

----------------------------------

# Conclusion

Before we make our first transfer, let's check that that the account is actually funded by asking the cluster for our balance!
