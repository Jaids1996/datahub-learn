---
description: Learn how to build a nameservice application (continued)
---

# The Keeper

## The Keeper

The main core of a Cosmos SDK module is a piece called the `Keeper`. It is what handles interaction with the store, has references to other keepers for cross-module interactions, and contains most of the core functionality of a module.

### Keeper Struct <a id="keeper-struct"></a>

To start your SDK module, define your `nameservice.Keeper` in the `./x/nameservice/keeper/keeper.go` file. Defined in this generated file are a few extra items that we will not cover at this time, for this reason we will start by clearing the `keeper.go` file in favor of following this tutorial.

```javascript
package keeper

import (
    "github.com/cosmos/cosmos-sdk/codec"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"
)

// Keeper maintains the link to data storage and exposes getter/setter methods for the various parts of the state machine
type Keeper struct {
    CoinKeeper types.BankKeeper

    storeKey  sdk.StoreKey // Unexposed key to access store from sdk.Context

    cdc *codec.Codec // The wire codec for binary encoding/decoding.
}
```

A couple of notes about the above code:

* Two `cosmos-sdk` packages and `types` for your application are imported:
  * [`codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec) - the `codec` provides tools to work with the Cosmos encoding format, [Amino](https://github.com/tendermint/go-amino).
  * [`types` \(as sdk\)](https://godoc.org/github.com/cosmos/cosmos-sdk/types) - this contains commonly used types throughout the SDK.
  * `types` - it contains `BankKeeper` you have defined in previous section. 
* The `Keeper` struct. In this keeper there are a couple of key pieces:
  * `types.BankKeeper` - This is an interface you had defined previous section to use `bank` module. Including it allows code in this module to call functions from the `bank` module. The SDK uses an [object capabilities](https://en.wikipedia.org/wiki/Object-capability_model) approach to accessing sections of the application state. This is to allow developers to employ a least authority approach, limiting the capabilities of a faulty or malicious module from affecting parts of state it doesn't need access to.
  * [`*codec.Codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec#Codec) - This is a pointer to the codec that is used by Amino to encode and decode binary structs.
  * [`sdk.StoreKey`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#StoreKey) - This is a store key which gates access to a `sdk.KVStore` which persists the state of your application: the Whois struct that the name points to \(i.e. `map[name]Whois`\).

### Getters and Setters <a id="getters-and-setters"></a>

Now it is time to add methods to interact with the stores through the `Keeper`. First, add a function to set the Whois a given name resolves to:

```javascript
// Sets the entire Whois metadata struct for a name
func (k Keeper) SetWhois(ctx sdk.Context, name string, whois types.Whois) {
    if whois.Owner.Empty() {
        return
    }
    store := ctx.KVStore(k.storeKey)
    store.Set([]byte(name), k.cdc.MustMarshalBinaryBare(whois))
}
```

In this method, first get the store object for the `map[name]Whois` using the the `storeKey` from the `Keeper`.

> _NOTE_: This function uses the [`sdk.Context`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#Context). This object holds functions to access a number of important pieces of the state like `blockHeight` and `chainID`.

Next, you insert the `<name, whois>` pair into the store using its `.Set([]byte, []byte)` method. As the store only takes `[]byte`, we use the Cosmos SDK encoding library called Amino to marshal the `Whois` struct to `[]byte` to be inserted into the store.

If the owner field of a `Whois` is empty, we do not write anything to the store, as all names that exist must have an owner.

Next, add a method to resolve the names \(i.e. look up the `Whois` for the `name`\):

```javascript
// Gets the entire Whois metadata struct for a name
func (k Keeper) GetWhois(ctx sdk.Context, name string) types.Whois {
    store := ctx.KVStore(k.storeKey)
    if !k.IsNamePresent(ctx, name) {
        return types.NewWhois()
    }
    bz := store.Get([]byte(name))
    var whois types.Whois
    k.cdc.MustUnmarshalBinaryBare(bz, &whois)
    return whois
}
```

Here, like in the `SetName` method, first access the store using the `storeKey`. Next, instead of using the `Set` method on the store key, use the `.Get([]byte) []byte` method. As the parameter into the function, pass the key, which is the `name` string casted to `[]byte`, and get back the result in the form of `[]byte`. We once again use Amino, but this time to unmarshal the byteslice back into a `Whois` struct which we then return.

If a name currently does not exist in the store, it returns a new `Whois`, which has the minimumPrice initialized in it.

Next, add a method to delete the names:

```javascript
// Deletes the entire Whois metadata struct for a name
func (k Keeper) DeleteWhois(ctx sdk.Context, name string) {
    store := ctx.KVStore(k.storeKey)
    store.Delete([]byte(name))
}
```

Here, like in the `SetName` method, first access the store using the `storeKey`. Next, we delete the name from the store using its `.Delete([]byte)` method. As the store only takes `[]byte`, the `name` string is casted to `[]byte` while passing it as a function parameter.

Now, we add functions for getting specific parameters from the store based on the name. However, instead of rewriting the store getters and setters, we reuse the `GetWhois` and `SetWhois` functions. For example, to set a field, first we grab the whole `Whois` data, update our specific field, and put the new version back into the store.

```javascript
// ResolveName - returns the string that the name resolves to
func (k Keeper) ResolveName(ctx sdk.Context, name string) string {
    return k.GetWhois(ctx, name).Value
}

// SetName - sets the value string that a name resolves to
func (k Keeper) SetName(ctx sdk.Context, name string, value string) {
    whois := k.GetWhois(ctx, name)
    whois.Value = value
    k.SetWhois(ctx, name, whois)
}

// HasOwner - returns whether or not the name already has an owner
func (k Keeper) HasOwner(ctx sdk.Context, name string) bool {
    return !k.GetWhois(ctx, name).Owner.Empty()
}

// GetOwner - gets the current owner of a name
func (k Keeper) GetOwner(ctx sdk.Context, name string) sdk.AccAddress {
    return k.GetWhois(ctx, name).Owner
}

// SetOwner - sets the current owner of a name
func (k Keeper) SetOwner(ctx sdk.Context, name string, owner sdk.AccAddress) {
    whois := k.GetWhois(ctx, name)
    whois.Owner = owner
    k.SetWhois(ctx, name, whois)
}

// GetPrice - gets the current price of a name
func (k Keeper) GetPrice(ctx sdk.Context, name string) sdk.Coins {
    return k.GetWhois(ctx, name).Price
}

// SetPrice - sets the current price of a name
func (k Keeper) SetPrice(ctx sdk.Context, name string, price sdk.Coins) {
    whois := k.GetWhois(ctx, name)
    whois.Price = price
    k.SetWhois(ctx, name, whois)
}

// Check if the name is present in the store or not
func (k Keeper) IsNamePresent(ctx sdk.Context, name string) bool {
    store := ctx.KVStore(k.storeKey)
    return store.Has([]byte(name))
}
```

The SDK also includes a feature called an `sdk.Iterator`, which returns an iterator over all the `<Key, Value>` pairs in a specific spot in a store. We will add a function to get an iterator over all the names that exist in the store.

```javascript
// Get an iterator over all names in which the keys are the names and the values are the whois
func (k Keeper) GetNamesIterator(ctx sdk.Context) sdk.Iterator {
    store := ctx.KVStore(k.storeKey)
    return sdk.KVStorePrefixIterator(store, []byte{})
}
```

The last piece of code needed in the `./x/nameservice/keeper/keeper.go` file is a constructor function for `Keeper`:

```javascript
// NewKeeper creates new instances of the nameservice Keeper
func NewKeeper(cdc *codec.Codec, storeKey sdk.StoreKey, coinKeeper types.BankKeeper) Keeper {
    return Keeper{
        cdc:        cdc,
        storeKey:   storeKey,
        CoinKeeper: coinKeeper,
    }
}
```

Next, its time to move onto describing how users interact with your new store using `Msgs` and `Handlers`.

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

