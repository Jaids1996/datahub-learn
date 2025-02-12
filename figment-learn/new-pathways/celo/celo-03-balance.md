Now that we have our account created, wouldn’t it be nice to keep track of our cUSD and CELO balances? In this step, we will examine how we can do just that!

{% hint style="info" %}
The Celo blockchain has two native assets, **CELO** (CELO) and the **Celo Dollar** (cUSD). Both of these assets implement the `ERC20` token standard from Ethereum. The CELO asset is managed by the CELO smart contract and Celo Dollars is managed by the cUSD contract. 
{% endhint %}

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/celo/balance.ts`, implement the **balance** function. You must replace any instances of `undefined` with working code to accomplish this.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...
  try {
    const { address } = req.body;
    const url = getSafeUrl();
    const kit = newKit(url);

    const goldtoken = undefined;
    const celoBalance = undefined;
    
    const stabletoken = undefined;
    const cUSDBalance = undefined;


    res.status(200).json({ 
        attoCELO: celoBalance.toString(), 
        attoUSD: cUSDBalance.toString() 
    });
  }
//...
```

**Need some help?** Check out these links
* [**We can access the CELO contract via the SDK with kit.contracts.getGoldToken()**](https://docs.celo.org/developer-guide/contractkit/contracts-wrappers-registry#interacting-with-celo-and-cusd)
* [**We can access the cUSD contract with kit.contracts.getStableToken()**](https://docs.celo.org/developer-guide/contractkit/contracts-wrappers-registry#interacting-with-celo-and-cusd)
* [**Reading from Alfajores**](https://docs.celo.org/developer-guide/start/hellocelo#reading-alfajores)

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
// solution
//...
  try {
    const { address } = req.body
    const url = getSafeUrl();
    const kit = newKit(url);

    const goldtoken = await kit.contracts.getGoldToken();
    const celoBalance = await goldtoken.balanceOf(address);

    const stabletoken = await kit.contracts.getStableToken();
    const cUSDBalance = await stabletoken.balanceOf(address);

    res.status(200).json({ 
        attoCELO: celoBalance.toString(), 
        attoUSD: cUSDBalance.toString() 
    })
  }
//...
```

**What happened in the code above?**

* First, we create a new `kit` instance.
* Next, we call the `getGoldToken` method of the `contracts` module to access CELO contract, then providing the input address to the `balanceOf` method, returning the balance of **CELO** token.
* Next, we call the `getStableToken` method of the `contracts` module to access the cUSD contract, then providing the input address to the `balanceOf` method, returning the balance of **cUSD** token.

{% hint style="tip" %}
The amount returned by these calls is denominated in **aCELO** and **acUSD**, which stands for "attoCELO" and "attocUSD" - representing [eighteen decimal places](https://en.wikipedia.org/wiki/Atto-). So to convert it to **CELO** and **cUSD** you'll need to divide it by 10**18 💪
{% endhint %}

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route. Click on **Check Balance** and you should see the balance displayed on the page:

![](../../../.gitbook/assets/pathways/celo/celo-balance.gif)

-----------------------------

# Conclusion

Querying the balance information is fun, but being able to submit transactions and change the state of a blockchain is even better! In the next step, we will dive deeper and submit our first transactions on Celo.
