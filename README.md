# Web3: Webpacked

This project is a drop-in solution for single-page Ethereum dApps. It's a [webpacked](https://webpack.js.org/) library consisting of:

- A robust management framework for the global `web3` object injected into browsers by [MetaMask](https://metamask.io/), [Trust](https://trustwalletapp.com/), etc. The framework exposes an instantiated [web3.js](https://web3js.readthedocs.io/en/1.0/) instance, keeps variables such as the current network and default account up-to-date, and fires customizable handlers when key events occur.

- Generic utility functions that fetch Ether and ERC20 balances, sign data, format [Etherscan](https://etherscan.io/) links, expose npm packages, etc.

- A fully managed solution for sending transactions that abstracts away from common annoyances like estimating gas usage and fetching current gas prices.

## Example Projects
Open a PR to add your project to this list!

## Installation

### Script Tag

Include the [minified bundle](./dist/web3Webpacked.min.js) (820 KiB) in your source code:

```html
<script src="js/web3Webpacked.min.js"></script>
```

This binds the library to the `window` as `w3w`.

### NPM
If you'd like to roll your own webpack solution, you can use the npm package:

```
npm install web3-webpacked
```

```javascript
const w3w = require('web3-webpacked')
```

In either case, to initialize the package:

```javascript
window.addEventListener('load', () => {
  console.log('Initializing web3 upon page load.')
  w3w.initializeWeb3(config)
})
```

If you don't need `web3` functionality immediately on page load, you can initialize the package later:

```javascript
if (document.readyState === 'complete') {
  console.log('Initializing web3 after page load.')
  w3w.initializeWeb3(config)
}
```

See [Config Options](#config-options) for more instructions on what to include in the optional `config` variable, or [Usage](#usage) to jump right in.

## Config Options
The following options can be set in the `config` variable passed to `initializeWeb3`.

### `handler`
- `Object` Up to five handlers triggered on various events.
  - `noWeb3Handler: function()` Triggered when no injected `window.web3` instance was found. This means that the user's browser does not have web3 support. Will also be triggered if your code calls `initializeWeb3` before web3 injection occurred (in which case you should re-read the [Installation](#installation) instructions).
  - `web3Ready: function()` Triggered when the `web3js` instance is available to be fetched with `getWeb3js`, and the current network/accounts can be fetched with the their respective getters.
  - `web3ErrorHandler: function([error])` Triggered when there was an error communicating with the Ethereum blockchain.
  - `web3NetworkChangeHandler: function([networkId, oldNetworkId])` Triggered on network changes.
  - `web3AccountChangeHandler: function([account, oldAccount])` Triggered on default account changes.

### `supportedNetworks`
- `Array` of `Numbers` Enforces that the injected `web3` instance is connected to a particular network. If the detected network id is not in the passed list, `web3ErrorHandler` will be triggered with an `UnsupportedEthereumNetworkError` error, and `web3` functionality will be removed. Supported network ids are: `1`, `3`, `4`, and `42`.

### Default Values
The default `config` values are below. You are encouraged to customize these!

```javascript
let config = {
  handlers: {
    // Prompt the user to e.g. install MetaMask or download Trust
    noWeb3Handler: () => {
      console.error('No web3 instance detected.')
    },
    // Check blockchain-dependent data
    web3Ready: () => {
      console.log('web3 initialized.')
    },
    // Notify the user of error, deal with unsupported networks
    web3ErrorHandler: (error) => {
      if (error.name === networkErrorName) {
        console.error(error.message)
      } else {
        console.error(`web3 Error: ${error}`)
      }
    },
    // Notify the user that they have switched networks, potentially re-instatiate smart contracts
    web3NetworkChangeHandler: (networkId, oldNetworkId) => {
      console.log(`Network switched from ${oldNetworkId} to ${networkId}.`)
    },
    // Notify the user that they have switched accounts, update balances
    web3AccountChangeHandler: (account, oldAccount) => {
      if (account === null) {
        console.log('No account detected, a password unlock is likely required.')
      } else {
        console.log(`Primary account switched from ${oldAccount} to ${account}.`)
      }
    }
  },
  pollTime: 1000, // 1 second
  supportedNetworks: [1, 3, 4, 42] // mainnet, ropsten, rinkeby, kovan
}
```

## Usage
- `w3w.initializeWeb3([config])`: Initialize web3 in your project. See above for more details.
- `w3w.getWeb3js()`: Returns a [web3js](https://web3js.readthedocs.io/en/1.0/) instance (web3@1.0.0-beta.34).
- `w3w.getAccount()`: Returns the current default account.
- `w3w.getNetworkId()`: Returns the current network id as a `Number`. Possible values: `1`, `3`, `4`, or `42`.
- `w3w.getNetworkName([networkId])`: Returns the name of a network (defaults to the current network). Possible values: `Mainnet`, `Ropsten`, `Rinkeby`, or `Kovan`.
- `w3w.getNetworkType([networkId])`: Returns the type of a network (defaults to the current network). Possible values: `PoW` or `PoA`.
- `w3w.getContract(ABI[, address, options])`: Returns a web3js Contract object.
- `w3w.getBalance([account, format])`: Returns the balance of an Ethereum address (defaults to the current account).
- `w3w.getERC20Balance([ERC20Address, account])`: Returns the token balance of an Ethereum address (defaults to the personal account) for any ERC20. Decimals are read from the smart contract.
- `w3w.toDecimal(number, decimals)`: number must be a `String`. Returns a decimalized version of the number as a `String`. Helpful when converting e.g. token balances from their `uint256` state in an Ethereum smart contract to actual balances.
- `w3w.fromDecimal(number, decimals)`: number must be a `String`. The opposite of `w3w.toDecimal`. Converts the number to an expanded form.
- `w3w.sendTransaction(method, handlers)`: An all-in-one function that manages the entire transaction sending flow. Ensures that function call won't fail given the current state of the network, that the sender has enough ether to cover the gas costs of the transaction, and calls `handlers` appropriately. `handlers` is an `Object` that must include an `error` handler, as well as optional `transactionHash`, `receipt`, and `confirmation` handlers. These correspond to [emitted web3js events](https://web3js.readthedocs.io/en/1.0/web3-eth.html#eth-sendtransaction-return).
- `w3w.signPersonal(message)`: Signs a message with the current default account per [this article](https://medium.com/metamask/the-new-secure-way-to-sign-data-in-your-browser-6af9dd2a1527). Returns the signing address, message hash, and signature. The returned signature is guaranteed to have originated from the returned address.
- `w3w.signTypedData(typedData)`: Signs typed data with the current default account per [this article](https://medium.com/metamask/scaling-web3-with-signtypeddata-91d6efc8b290). Returns the signing address, message hash, and signature. The returned signature is guaranteed to have originated from the returned address.
- `w3w.etherscanFormat(type, data[, networkId])`: Returns an [Etherscan](https://etherscan.io/) link to a given `transaction`, `address`, or `token` (defaults to the current network).
- `w3w.networkErrorName`: The name of the error thrown when the injected web3 instance is on an unsupported network.
- `w3w.libraries.`
  - `eth-sig-util`: Exposes the [eth-sig-util](https://github.com/MetaMask/eth-sig-util) package.
  - `ethereumjs-util`: Exposes the [ethereumjs-util](https://github.com/ethereumjs/ethereumjs-util) package.


## Notes
- To ensure that your code is accessing the most up-to-date variables, be sure not to hard code values like the `web3js` instance, the default `account`, the current `networkId`, etc. Instead, call functions like `w3w.getWeb3js()` on demand, whenever you need a `web3js` instance.

- `web3-webpacked` is forced to be opinionated, and has integrated web3js in lieu of possible alternatives like [ethjs](https://github.com/ethjs/ethjs). In the future, it's possible that two branches will be maintained, with web3js and ethjs compatibility respectively. There is also an argument to be made for letting users build `web3-webpacked` themselves with arbitrary versions of these web3 APIs (how exactly this would look is TBD). If any of this is of interest, please submit an issue with your ideas/comments.
