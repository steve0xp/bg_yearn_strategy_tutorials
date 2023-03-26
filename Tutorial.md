# 🏗 scaffold-eth | 🏰 BuidlGuidl

## 🚩 **Challenge X: Yearn Strategy General Info**

This challenge is focused on guiding students through the following:

- What a Yearn Strategy is
- The typical ins and outs of a Yearn strategy to be aware of that Yearn v2 Vaults take with their actual strategies
- The approach taken to writing this yearn strategy (how @steve0xp got into writing this yearn strategy where he is focused on getting it to ape.tax)
- Writing this yearn strategy and getting experience dealing with some types of the basics.

**This is the first part of 2 challenges focused on writing Yearn strategies.**

1. First challenge - create a super simple strategy to get used to the ins and outs of the `harvest` tx flow.
2. Second challenge - create a strategy atop of Mellow Protocol && Gearbox Protocol w/ tests!

---
### **🚨 DeFi Sub-Branch Context / Disclaimers**

This challenge is part of the DeFi Sub-Branch, where students are invited into writing smart contracts that incorporate actual DeFi protocols. The sub-branch projects are ever-expanding as DeFi is too. The goal of the Defi sub-branch projects/tutorials are to provide students tutorials that foster self-learning in actual DeFi protocols, guide you through some of the basics of integrating with specific protocols, and other useful tips to increase competence to be potentially hireable as an intermediate developer. 

> ❗️ NOTE: **Students taking on this challenge will be expected to embrace self-learning AND it is recommended that they have completed all beginner SRE challenges or show competency elsewhere.** 

<details markdown='1'><summary>👩🏽‍🏫 More Disclaimers </summary>

This requirement is simply because tying into other protocols is powerful, and with that comes a lot of responsibility when deploying contracts that may end up holding people's actual funds. These tutorials don't do that but they start to show paths for people to build possible fund-holding code. It is key to know what you are doing.

As well, self-learning is required because tying into DeFi protocols and actual professional crypto projects do not guide developers, step-by-step. If you are new to trying to actually plug into a professional project, no worries! We'll touch on some helpful tips as you sort out what the 'norm' is when going through this process. These tips will be specific to each protocol that we are integrating into and learning about.

</details>

---
## Yearn Strategy Intro Challenge

### Repo Setup
TODO: Change wording here to suit this challenge

To start the project, clone the repo to your local machine using the following CLI commands:

1. Clone the repo onto your local machine and install the submodules: `git clone --recursive <repo link>`

   > NOTE: If you have not installed the submodules, probably because you ran `git clone <repo link>` instead of the CLI command in step 1, you may run into errors when running `forge build` since it is looking for the dependencies for the project. `git submodule update --init --recursive` can be used if you clone the repo without installing the submodules.

2. Install forge on your machine if you have not already: `forge install`

> NOTE: If you need to download the latest version of foundry, just run `foundryup` 

3. Build the project and make sure everything compiles: `forge build`

#### Testing Against Local Mainnet Fork
To run unit tests against a non-persistent local mainnet fork, first make sure you have a `.env` file set up at the root (follow `.env.example` format) and populate the `MAINNET_RPC` variable like so:

`MAINNET_RPC=<insert ETH RPC URL here>`

Make sure you are using latest version of foundry, so that it auto-sources `.env`, otherwise run (while in the root directory): `source .env`

CLI command: `forge test -vvv`

You can also test individual contracts with the following:
- `forge test --match-contract Kernel -vvv`
- Where `Kernel` is the name of the contract

#### Deploying contracts on Local Mainnet Fork
To deploy the code onto a persistent mainnet fork, first make sure you have a `.env` file set up at the root (follow `.env.example` format):

- ```MAINNET_RPC=<insert ETH RPC URL here>```
- ```FORK_URL=<insert your localhost fork url - usually http://localhost:8545>```
- ```PRIVATE_KEY=<should be the first private key supplied by anvil>```

#### Deploying the base protocol contracts

1. run `source .env` in the base folder
2. open a new terminal and run `anvil --fork-url $MAINNET_RPC`
3. copy the first private key from the given private keys into `.env`
4. run `source .env` to update the `PRIVATE_KEY`
5. open a new terminal and on the base folder run 
- ```forge script scripts/DeployProtocol.s.sol:DeployProtocol --fork-url $FORK_URL --broadcast --json  --private-key $PRIVATE_KEY```

- To find the deployed contracts' addresses, head over to `broadcast` folder and find the latest run log.

🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨
---

### Simple Yearn Strategy

To kick things off, we'll start by going over the yearn strategy tutorial that Yearn core dev, @cdalton, has gone over in this video. Here he covers making a yearn strategy where we pull DAI from the DAIVault in Yearn and deploy it in a lending pool in Compound. If you're not familiar with Compound, they are an OG lending protocol where users can be lenders and borrowers of whitelisted tokens (deemed legitimate through Compound governance).

This tutorial goes over:

- Building a strategy using foundry testing framework
- Video touches on testing with foundry a bit

![image](./images/HowYearnWorks.png)

- For each yVault there are multiple strategies. Example in the above image: DAI is distributed to different strategies where certain amounts have allocations to it.

**Accounting VIP Details**
It's key to know that the strategies take on `debt` from the vaults. AKA some amount of DAI is given to the strategy from the vault. So the strategies below would have some `debt` w/ specified amounts.

Let's touch on the strategy flows:

**Strategy Flow #1**

_Don't worry about the bots, we use OP, a template pattern that extends from the template_

`harvest()` is called by a `keep3r` which does the following:
- `prepareReturn(debtOutstanding)` : decelaring accounting losses and gains since the last harvest. `debtOutstanding` is the amount of wantToken that the vault wants back. So within `prepareReturn()` there will be internal calls to liquidate, if needed, some wantToken to obtain the requested amount of `debt` back to the vault from the respective strategy. This is a function that exists in every strategy.
- `adjustPosition(debtOutstanding)` : the inverse of the above, investing any excess `wantToken` into the respective strategy. In this case DAI.

![image](./images/StrategyFlow1.png)


**Strategy Flow #2**

User withdraws from the vault which calls withdraw from the strategy. This further calls a function called `liquidatePosition(amountNeeded)`. If not enough funds then losses are declared (accounting side of things).

`withdraw()`

![image](./images/StrategyFlow2.png)

**Why Build with Foundry**

- 10x faster WITH fuzzing which is good for catching edge cases (rounding, etc.)
- Fuzzing
- Console.log
- Solidity so less context switching vs testing with Brownie in this case

### Diving into the code

So this is started from a foundry mix that @storming0x made [here](LINK).

The tutorial goes over the changes made that you can will implement into your challenge! *As always, solutions are made but don't peek!!!

In strategy.sol

- import OZ library for math // TODO: not sure we need this anymore actually w/ solidity v8.0
- import CDAI as a constant
   - import ICToken.sol because we are building a compound-DAI strategy. // TODO: add a note stating that this is done through going and getting the interface itself from the repo itself (Include the interface itself here so the tutorial doesn't break but point to where you got it from so students understand how to get interfaces for projets).
    - CDAI will be the pool we're deploying into.

Line of code:

````
import "@openzeppelin/contracts/utils/math/Math.sol";

import "./interfaces/Compound/ICToken.sol";

// inside the contract itself (below)
ICToken internal constant cDAI = ICToken(0x5d3a536E4D6DbD6114cc1Ead35777bAB948E3643);
````

CToken interface details:

function 1: mint()
- amount of DAI is given in, and it gives you the amount of CDAI it's worth.

redeem():
- amount of CDAI that you want to burn and it gives you back however much DAI it's worth.

exchangeRateStored():
- when valuing the CDAI position we have, we multiply the CDAI position by this exchange rate and it gives the value of DAI in CDAI.

### code

In the constructor:

We need to first approve the amount of DAI that we are putting into this CDAI pool. So we're giving the CDAI pool permission to handle our tokens (DAI).

```
want.approve(address(cDAI), type(uint256).max)
```

Let's name the strategy. There is a typical convention as Facu had pointed out in past yearn strategy tutorial videos.

### `estimatedTotalAssets()` Implementation

Cool, now we'll go into the next typical function, `estimatedTotalAssets()`

This function is called throughout the general "flow" of a strategy, in particular for accounting purposes. We need to obtain an estimate of total assets in the form of `wantToken`. Write up the implementation for this function so it accounts for the `wantToken` within the strategy itself AND the `wantToken` of the assets within the strategy itself. _hint: don't forget to check if there are any scaling factors applied in your conversions._

<details markdown='1'><summary>👩🏽‍🏫 Solution Code</summary>

As you can see, the exchangeRateStored() is **scaled up** by a factor of 18, so we must normalize the exchange rate by a factor of 18. Here you can see the `estimatedTotalAssets` are the sum of: `wantToken` in the strategy, `assets` converted to `wantToken` and scaled down properly by `1e18`
```
return want.balanceOf(address(this)) + cDAI.balanceOf(address(this)) * cDAI.exchangeRateStored() / 1e18; 
```

</details>

### `prepareReturn()` Implementation

Great! Now we're going to look at the `prepareReturn()` function. It does two things:

1. Prepares `wantToken` to be returned
2. Takes care of accounting (profits and losses since last harvest), and specifies how much debt it can pay back of the `debtOutstanding` in the form of the return param `debtPayment`



The standard flow of things is to:

- calculate total assets and total debt in this strategy. We'll use the previous `estimatedTotalAssets()` function for the former && we can use `totalDebt` function from yearn. 

Write the implementation logic for this function such that it will return the proper params; `_profit, _loss, _debtPayment`

*Note that all will be in `wantToken`

As well, add in a stubbed out function call to a helper function called `withdrawSome()`. This helper function will eventually take care of liquidating enough `wantToken` already deployed in the strategy (CDAI pool) and attempt to get enough to repay the debt being requested. More on this helper function later though.

_hint: you'll want to write conditional logic for when there is a profit vs when there is not_

<details markdown='1'><summary>👩🏽‍🏫 Solution Code</summary>

```
uint256 _totalAssets = estimatedTotalAssets();
uint256 _totalDebt = vault.strategies(address(this)).totalDebt;

if(_totalAssets >= _totalDebt) {
    _profit = _totalAssets - _totalDebt;
    _loss = 0;
} else {
    _profit =0;
    _loss = _totalDebt - _totalAssets;
}

withdrawSome(_debtOutstanding + _profit); // _profit needs to be liquid in wantToken, so `withdrawSome()` takes care of that

uint256 _liquidWant = want.balanceOf(address(this));

// enough to pay profit (partial or full) only
if(_liquidWant <= profit) {
    _profit = _liquidWant;
    _debtPayment = 0;
// enough to pay for all profit and _debtOutstanding (partial or full)

}
```

</details>

### `adjustPosition()` Implementation

The focus of this function is to invest excess `wantToken` into the respective strategy. In this case we're investing any excess DAI into the CDAI pool.

Write the implementation logic such that it invests any excess DAI, based on the `debtOutstanding` param, into the CDAI pool.

_hint: make sure that there are checks implemented to ensure that the excess DAI was truly invested (see ICToken interface return values 😉)_

<details markdown='1'><summary>👩🏽‍🏫 Solution Code</summary>

```
uint256 _daiBal = want.balanceOf(address(this));

if(_daiBal > _debtOutstanding) {
    uint256 _excessDai = _daiBal - _debtOutstanding;
// CToken function mint() gives back a uint256 == 0 if it is successful.
    uint256 _status = cDAI.mint(_excessDai);
    assert(_status == 0);
}

```

</details>

### `withdrawSome()` Implementation

Recall from before that this was the stubbed out function to actually liquidate CDAI to DAI (`wantToken`) as needed.

<details markdown='1'><summary>👩🏽‍🏫 Solution Code</summary>

Recall: passed in param to this function is in DAI, and we need to convert that to CDAI because `redeem` takes a CDAI amount. So multiply by 1e18 and divide by exchange rate!

```
function withdrawSome(uint256 _amountNeeded) internal {
    uint256 _cDaiToBurn =Math.min(_amountNeeded * 1e18 / cDAI.exchangeRateStored());

    uint256 _status = cDAI.redeem(_cDaiToBurn);
    assert(_status == 0);
}   
```

</details>

Nice, now we'll move onto `liquidatePosition()`

### `liquidatePosition()` Implementation

Remember that this is called during the flow "withdraw"

First calculate how much DAIBal we have already. If we have enough daiBal to pay things back then we'll return the amountNeeded accordingly - we'd report 0 loss then.

Otherwise we're going to call withdrawSome(), get more DAI from our strategy, and then check if we have enough.

<details markdown='1'><summary>👩🏽‍🏫 Solution Code</summary>

```
function liquidatePosition(uint256 _amountNeeded) internal override returns (uint256 _liquidateAmount, uint256 _loss) {
    uint256 _daiBal = want.balanceOf(address(this));

    if (_daiBal >= _amountNeeded) {
        return (_amountNeeded, 0);
    }

    withdrawSome(_amountNeeded);

    _daiBal = want.balanceOf(address(this));
    if (_amountNeeded > _daiBal) {
        _liquidatedAmount = _daiBal;
        _loss = _amountNeeded - _daiBal;
    } else {
        _liquidateAmount = _amountNeeded;
    }
}
```

</details>

At this point we have all the basic functions written out! Congrats!

Now we'll check that your code actually works. Luckily we're leveraging the working foundry mix from Yearn protocol themselves. Part of their developer relations side of things is to onboard new strategists easily by having them only need to worry about risk vectors associated to their strategy itself. They don't need to worry about the harvest flows, interactions with the vaults, etc. typically! So that means that running `make test` will run the typical integrations tests that are needed to ensure a good working strategy with the rest of the yearn v2 vaults architecture is sound!

_NOTE: `make test` is just a script that ultimately runs a series of `forge test` and you can dig into it more by investigating the yearn foundry mix repo itself with its `package.json`_



---
## References and Context for Those Interested in How this Tutorial was Made

The DeFi Level 3000 repo is always growing. If you are interested in contributing by writing tutorials that educating others in integrating with a different protocol please open an issue, or even fork the repo and submit a PR or draft PR!

For inspiration and/or reference, we've listed out how this strategy came to be, it was very organic!
### **💡🚶🏻‍♂️ Steps taken when Starting this Yearn Strategy Itself**

A typical professional crypto project will have developer docs to help onboard devs into the respective tech stack. The approach that was taken when writing this yearn strategy was to read through the docs, and jump in to the communication channels offered from the protocol. 

@steve0xp simply wanted to learn how to write yearn strategies, and was fortunate enough to have met some yearn contributors at past conferences and throughout online async communications (being part of their discord, writing threads on their project, etc.).

In the case of getting involved with writing strategies, he simply went onto the Yearn Discord, chatted about building yearn strategies, and was pointed to the [Yearn General Telegram Group Chat](https://t.me/yearnfinance) && eventually the gated Telegram group chat, YFI Boarding School. Here he met other strategists, and core strategists who actually deployed strategies with real funds in them.

> **@steve0xp wrote threads outlining his journey writing this Yearn Strategy, we heavily recommend checking out the respective thread highlighted through different parts of this tutorial.**

>We'll list them out all here now though for easier reference. These offer a general landscape of this tutorial's contents. We will use his journey && strategy as the basis of this tutorial.

<details markdown='1'><summary>👩🏽‍🏫 DRAFT THREADS - TODO: finalize these links once the threads are posted. </summary>

1. [Getting started, conceptualization && initialization](https://typefully.com/t/X7kgQ4J)
2. [Troubleshooting with other devs && discussions](https://typefully.com/t/7viHbCb)
3. [Q/C through writing unit tests](tbd)
4. [Deployment && getting actual dollars into the strategy from yearn vaults](tbd)

</details>

---

### README from @storming0x foundry_strategy_mix

The following are the README details found typically in the foundry_strategy_mix. You would use this to start a brand new strategy (assuming you are building with foundry). The repo is updated periodically so make sure to pull the most recent ones and direct any questions to the Yearn team if you are having any troubles!

<details markdown='1'><summary>👩🏽‍🏫 README.md from @storming0x foundry_strategy_mix </summary>

# Yearn Strategy Foundry Mix

## What you'll find here

- Basic Solidity Smart Contract for creating your own Yearn Strategy ([`Strategy.sol`](src/Strategy.sol))

- Configured github template with Foundry framework for starting your yearn strategy project.

- Sample test suite. ([`tests`](src/test/))

## How does it work for the User

Let's say Alice holds 100 DAI and wants to start earning yield % on them.

For this Alice needs to `DAI.approve(vault.address, 100)`.

Then Alice will call `Vault.deposit(100)`.

Vault will then transfer 100 DAI from Alice to itself, and mint Alice the corresponding shares.

Alice can then redeem those shares using `Vault.withdrawAll()` for the corresponding DAI balance (exchanged at `Vault.pricePerShare()`).

## Installation and Setup

1. To install with [Foundry](https://github.com/gakonst/foundry).

2. Fork this repository (easier) or create a new repository using it as template. [Create from template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template)

3. Clone your newly created repository recursively to include modules.

```sh
git clone --recursive https://github.com/myuser/foundry-yearn-strategy

cd foundry-yearn-strategy
```

NOTE: if you create from template you may need to run the following command to fetch the git submodules (.gitmodules for exact releases) `git submodule init && git submodule update`

4. Build the project.

```sh
make build
```

5. Sign up for [Infura](https://infura.io/) and generate an API key and copy your RPC url. Store it in the `ETH_RPC_URL` environment variable.
NOTE: you can use other services.

6. Use .env file
  1. Make a copy of `.env.example`
  2. Add the values for `ETH_RPC_URL`, `ETHERSCAN_API_KEY` and other example vars
     NOTE: If you set up a global environment variable, that will take precedence.

7. Run tests
```sh
make test
```

## Basic Use

To deploy the demo Yearn Strategy in a development environment:

TODO

## Implementing Strategy Logic

[`src/Strategy.sol`](contracts/Strategy.sol) is where you implement your own logic for your strategy. In particular:

- Create a descriptive name for your strategy via `Strategy.name()`.
- Invest your want tokens via `Strategy.adjustPosition()`.
- Take profits and report losses via `Strategy.prepareReturn()`.
- Unwind enough of your position to payback withdrawals via `Strategy.liquidatePosition()`.
- Unwind all of your positions via `Strategy.exitPosition()`.
- Fill in a way to estimate the total `want` tokens managed by the strategy via `Strategy.estimatedTotalAssets()`.
- Migrate all the positions managed by your strategy via `Strategy.prepareMigration()`.
- Make a list of all position tokens that should be protected against movements via `Strategy.protectedTokens()`.

## Testing

Tests run in fork environment, you need to complete [Installation and Setup](#installation-and-setup) step 6 to be able to run these commands.

```sh
make test
```
Run tests with traces (very useful)

```sh
make trace
```
Run specific test contract (e.g. `test/StrategyOperation.t.sol`)

```sh
make test-contract contract=StrategyOperationsTest
```
Run specific test contract with traces (e.g. `test/StrategyOperation.t.sol`)

```sh
make trace-contract contract=StrategyOperationsTest
```

See here for some tips on testing [`Testing Tips`](https://book.getfoundry.sh/forge/tests.html)

## Deploying Contracts

You can Deploy and verify your strategies if PRIV_KEY and ETHERSCAN_API_KEY are both set in the .env by using the command

```sh
make deploy
```

Before deploying, update the constructor-args variable within the Makefile to include any parameters applicable. Make sure to seperate each argument only by a space, no commas.

The deploy script is coded to deploy the "Strategy" contract within the Strategy.sol file. This can be updated by simply updating to src/YourContract.sol:YourContract.

## GitHub Actions

This template comes with GitHub Actions pre-configured. Your contracts will be linted and tested on every push to
`master` and `develop` branch.

Note though that to make this work, you must set your `INFURA_API_KEY` and your `ETHERSCAN_API_KEY` as GitHub secrets.

# Resources

- Yearn [Discord channel](https://discord.com/invite/6PNv2nF/)
- [Getting help on Foundry](https://github.com/gakonst/foundry#getting-help)
- [Forge Standard Lib](https://github.com/brockelmore/forge-std)
- [Awesome Foundry](https://github.com/crisgarner/awesome-foundry)
- [Foundry Book](https://book.getfoundry.sh/)
- [Learn Foundry Tutorial](https://www.youtube.com/watch?v=Rp_V7bYiTCM)


</details>