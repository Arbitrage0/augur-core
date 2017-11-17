# augur-core

[![Build Status](https://travis-ci.org/AugurProject/augur-core.svg)](https://travis-ci.org/AugurProject/augur-core)

Ethereum contracts for a decentralized prediction market platform.

## Installation

You should already have a system-wide installation of Python and it should be Python 2.7.6 or greater.

First install the dependencies, which include PyEthereum (the tool used to test Ethereum smart contracts from Python scripts) and the Solidity smart contract programming language:

```
npm install npx
npm install

sudo pip install -r requirements.txt
```

For PyEthereum to work properly you'll also need to install the 0.4.17 Solc compiler on your machine. Instructions for installing it in your environment can be found here:

[Solc 0.4.17 Installation Instructions](https://github.com/ethereum/solidity/releases/tag/v0.4.17)

On macOS you will need to use a
[virtualenv](https://python-guide-pt-br.readthedocs.io/en/latest/dev/virtualenvs/) or use a [homebrew](https://brew.sh/) Python to work around System Integrity Protection.

Now we can try running some tests to make sure our installation worked:

```
cd tests
pytest
```

## Docker

You may run augur-core using docker as follows:

### Build:

```
npm run docker:build
```

### Run:

```
npm run docker:test:unit
```

### Debug:

```
npm run docker:test:console
py.test -s tests/trading_tests.py
```

## General information about Augur

- [A Roadmap For Augur and What’s Next](https://medium.com/@AugurProject/a-roadmap-for-augur-and-whats-next-930fe6c7f75a)
- [Augur Master Plan](https://medium.com/@AugurProject/augur-master-plan-42dda65a3e3d)

## Terminology

[Augur Terminology](http://blog.augur.net/faq/all-terms/)

## Src Folders

The source code folders are split into six folders: extensions, factories, libraries, reporting, and trading. Splitting the folders this way allows for a separation of concerns and better modularity.
- source/contracts/extensions -- contains files that deal with market creation, bond validity, getting the market price and reporting fee in attoeth (10^-18 eth), creating order books, reporting messages, and getting free REP (reputationTokens)
- source/contracts/factories -- contains files that are constructors for universes, markets, reputation Tokens, reporting Periods/Windows, tradeable asset shareTokens, and topics for prediction.
- source/contracts/libraries -- contains data structures used elsewhere in the source code.
- source/contracts/reporting -- contains source files for the creation and manipulation of universes, markets, and various types of tokens (registration Tokens, stake Tokens, and reputation Tokens).
- source/contracts/trading -- contains source code files that handle user actions regarding orders, claims, topics, and other trade functions.

## Security model

The security model is based on the idea that reporters will want to invest in a market where honesty prevails, and will not trust fraudulent markets. For example, assume that there exist two almost identical universes: an honest universe, where the factually correct answer is the predominant answer, and a fraudulent universe, where incorrect answers are predominant in order to cheat the system. The REP in the fraudulent universe will approach 0, and the REP in the honest universe will approach the theoretical yield. This phenomenon occurs because reporters do not want to trade on markets in a dishonest universe. Therefore, they will not create markets in a dishonest universe. Ultimately, the REP holders will not receive any market fees.

In order to create a fraudulent universe, an attacker must fork the main universe. They will need to buy a large share of REP and stake it on the lie outcome, as nobody else will want to engage in a deceitful market. In principle, the attacker will lose up to 2.5 times the maximum profit made from a successful attack, and their REP will approach 0.

The reentrancy guard (libraries/ReentrancyGuard.sol) and controller (Controller.sol) contracts also help uphold security. One can also attack Ethereum contracts by creating a chain of wanted contract calls through a malicious user. The reentrancy guard protects against that by preventing someone from re-entering the contracts before they return. The controller contract serves to update Augur and stores a registry of whitelisted contract addresses. The controller also can perform administrative actions, such as transferring ownership and performing emergency stop functions. It can be operated in either the developer mode, where the developer updates contracts, or the decentralized mode, when people must upload contracts to make changes and therefore, the market creators/traders must cancel orders and shift to the new contract. For bugs found involving money or REP, contracts are locked / an escape hatch is enabled which allows withdrawal of shares [and cancelling orders] for a value half-way between the bid-ask spread at the time of the lock. In addition, another escape hatch is enabled, which allows withdrawal of any locked up REP.

## Testing

The tests directory (augur-core/tests) contain tests and test fixtures to test the various functionalities present in Augur, including trading, reporting, and wcl tests.
- conftest.py -- contains the class ContractFixture, which deals with caching compiled contracts, signatures, etc. as well as resetting the blockchain before each test.
- delegation_sandbox.py -- tests the delegator contract.
- sandbox.py -- used for testing miscellaneous Solidity behaviors
- reporting -- contains tests for reporting purposes.
- trading -- contains tests for trading purposes.
- solidity_test_helpers -- small contracts to help run tests.
- test_controller.py -- tests controller functionalities.
- test_mutex.py -- tests mutex functionalities.
- test_helpers.py -- tests the controller, safeMath, and assertNoValue macros.
- test_legacyRep.py -- tests the legacyRepTokens’ functionalities.
- utils.py -- contains useful functions for testing, such as conversion between different data types.
- wcl-in-python.py -- contains functions for making and taking various types of bids.
- wcl.txt -- explains tests for the various situations when filling a bid and filling an ask.

In order to run Augur’s test suite, run the following inside the tests directory:
```
cd augur-core/tests
pytest
```
This executes all the tests. To run a test individually, run the following:
```
pytest path/to/test_file.py -k 'name_of_test’
```

By default the trading WCL fuzz testing is not run, if you want to run it specifically do
```
export INCLUDE_FUZZY_TESTS=1
pytest trading/test_wcl_fuzzy.py
unset INCLUDE_FUZZY_TESTS
```

Or if you want to run all tests (including fuzzy):

```
export INCLUDE_FUZZY_TESTS=1
pytest
unset INCLUDE_FUZZY_TESTS
```

When writing tests, it is highly recommended to make use of the ContractFixtures class for ‘placeholder’ variables. Python’s unit testing framework comes handy here; encapsulate tests within functions that start with "test\_", and use “assert” statements when testing for certain values. Parameterized tests are recommended as well to test various possibilities and edge cases.

## Deployment

Solidity contract deployment is handled by `ContractDeployer.ts` and the wrapper programs located in `source/deployment`. This deployment framework allows for incremental deploys of contracts to a given controller (specified via a configuration option). This allows us to deploy new contracts without touching the controller, effectively upgrading in-place the deployed system.

### How to run a deployment

Deployment can be run in two modes, direct or docker. In direct mode the assumoption is that the entire system has been built (typescript and solidity), and there is a working nodejs environment locally. Furthermore, one must posses an account on the deployment target (e.g. Rinkeby testnet) which has enough ETH to cover the costs of the deployment. For the purposes of deploying to the testnets, those with access to the augur private testnet keys can deploy and update the existing contracts. For reference, these keys are stored in an encrypted git repository within the Augur Project's keybase team. If you need access -- please inquire within Discord.

All deployment commands can be managed through scripts described in package.json, and are accessible to `npm run` and `yarn`.

### Example Rinkeby Deployment
```
augur-core % npm run build # build and create compile artifacts
augur-core % RINKEBY_PRIVATE_KEY=$(cat path/to/keys/deploy_keys/rinkeby.prv) npm run deploy:rinkeby
```


### Example Updating augur-contracts
This deployment will generate artifacts in the ./output/contracts directory. To merge these changes into augur-contracts, there are scripts located in that repository. *When running deploymen locally this step is not done automatically*. To include these changes you can work from within your checked out augur-contracts.

```
augur-contracts % SOURCE="path/to/augur-core/output/contracts" BRANCH=master npm run update-contracts
augur-contracts % npm version patch
augur-contracts % git push && git push --tags
augur-contracts % npm publish
```

## Additional notes

There are no floats in the EVM.

All augur-core contracts use integers. So ether values in contracts will always be represented as integers whose value is in attoeth (aka: wei, 10^-18 ether).

## Information about the new reporting system

- [Flow diagram](https://pasteboard.co/1FcgIDWR2.png)
- [More in depth diagram](https://www.websequencediagrams.com/files/render?link=kUm7MBHLoO87M3m2dXzE)
- [Market object graph](https://pasteboard.co/1WHGfXjB3.png)

## Information about trading worst case loss (WCL)

- [Some notes on WCL/value at risk](https://github.com/AugurProject/augur-core/blob/master/tests/wcl.txt)
