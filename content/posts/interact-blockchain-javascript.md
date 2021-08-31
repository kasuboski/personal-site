---
title: "Interact with Blockchain Apps in Javascript"
date: 2021-08-30T21:11:59-05:00
draft: false
---

Interfacing with a blockchain can seem intimidating, but this dapp has a nice javascript sdk.

<!--more-->

## What's it do?
I made a script that will check your collateral positions in [Mirror Protocol](https://terra.mirror.finance/). Mirror is an app on the [Terra](https://www.terra.money/) blockchain that lets you trade stocks, but in reality they are tokens that try to stay pegged to the real life equivalent.

There's an option to short an asset in Mirror. This requires that you put up collateral as you are essentially borrowing an asset and promise to pay it back later. The collateral you put up needs to maintain a certain ratio or it will be liquidated to pay your debt.

If the asset you borrowed grows in value too fast you may need to add more collateral or close your position to not get liquidated. You can imagine you might want to know if you're getting close, before it happens, and while not checking constantly.

## Interacting with Mirror
Mirror provides a Javascript SDK at [Mirror.js](https://docs.mirror.finance/developer-tools/mirror.js). The documentation is a little lacking, but between reading the source code and the queries available on the actual [smart contract](https://docs.mirror.finance/contracts/mint#positions) you can find everything you need.

Once you have this SDK, interacting with Mirror and Terra is pretty simple. The entire process of checking if your collateral ratio is safe is shown below.

```javascript
async function checkPositions(address) {
  // find positions
  // if short look up price asset vs collateral
  // check that collateral ratio is safe
  const positions = await mirror.mint.getPositions(address);
  const short_positions = positions.positions.filter(pos => pos.is_short);
  const info = await Promise.all(short_positions.map(async pos => {
    const collateralPrice = await getCollateralPrice(pos.collateral.info.token.contract_addr);
    const assetPrice = await getAssetPrice(pos.asset.info.token.contract_addr);

    const collateralAmount = Number.parseInt(pos.collateral.amount);
    const assetAmount = Number.parseInt(pos.asset.amount);

    const collateralValue = collateralAmount * collateralPrice;
    const assetValue = assetAmount * assetPrice;

    const collateralRatio = collateralValue / assetValue;
    const minRatio = await getMinCollateralRatio(pos.asset.info.token.contract_addr);
    const safeRatio = minRatio + 0.5; // taken from mirror site
    const isSafe = collateralRatio > safeRatio;

    return {
      collateralPrice,
      assetPrice,
      collateralAmount,
      assetAmount,
      collateralValue, 
      assetValue,
      collateralRatio,
      isSafe,
    };
  }));

  return info;
}
```

All of the information we need has mappings in Javascript so it feels like calling a function. If there's anything not exposed there is also a GraphQL API. If you want to try it out you can find it on GitHub at [kasuboski/mirror-watch](https://github.com/kasuboski/mirror-watch). You just need to know a Terra address. It can be yours or someone else's since all data is public.

## Moving Forward
I already made a similar script for [Anchor Protocol](https://anchorprotocol.com/), which is also on Terra, at [kasuboski/anchor-watch](https://github.com/kasuboski/anchor-watch). There are already projects building notifications and auto repayment strategies. There are also projects that are trying to combine Mirror Assets to make a pseudo ETF. 

I started messing with [Terra Academy](https://academy.terra.money), where it walks you through building a smart contract in Rust. Maybe I'll end up doing something with that, but for now it's easy enough to interface just using the various APIs. No need to live on-chain.