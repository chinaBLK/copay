## Main Features

- Multiple wallet creation and management in-app
- Intuitive, multisignature security for personal or shared wallets
- Easy spending proposal flow for shared wallets and group payments
- [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) Hierarchical deterministic (HD) address generation and wallet backups
- Device-based security: all private keys are stored locally, not in the cloud
- Synchronous access across all major mobile and desktop platforms
- Payment protocol (BIP70-BIP73) support: easily-identifiable payment requests and verifiable, secure bitcoin payments
- Mnemonic (BIP39) support for wallet backups
- Paper wallet sweep support (BIP38)
- Push notifications (only available for ios and android versions)
- Customizable wallet naming and background colors
- Multiple languages supported


## Install For Development

Clone the repo and open it

Ensure you have [Node](https://nodejs.org/) installed

```sh
npm install
npm start
```

Visit [`localhost:3000`](http://localhost:3000/) to view the app.

> **Note:** This method should only be used for development purposes. When running Copay in a normal browser environment, browser extensions and other malicious code might have access to internal data and private keys. For production use, see the latest official [releases](https://github.com/bitpay/copay/releases/).

## Build Copay App Bundles

### Android

- Install Android SDK
- Run `make android`

### iOS

- Install Xcode 6.1 (or newer)
- Run `make ios-prod`

##### Notes for Xcode 7.0

###### ATS support

Before starting Copay from Xcode, add these lines to "Custom iOS Target Properties":

```
<key>NSAppTransportSecurity</key>
 <dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
 </dict>
```

![Example](http://i.stack.imgur.com/nGw3j.png)


App Transport Security (ATS) enforces best practices in the secure connections between an app and its back end. [Read complete documentation](https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html).

###### Invalid Bundle while submitting application

`iPad Multitasking support requires launch story board in bundle`

To fix this problem, add the following:

```
<key>UIRequiresFullScreen</key>
<string>YES</string>
```
###### Build settings, headers search path

Add this line to your Build Settings -> Header Search Paths -> Release

"$(OBJROOT)/UninstalledProducts/$(PLATFORM_NAME)/include"



### Windows Phone

- Install Visual Studio 2013 (or newer)
- Run `make wp8-prod`

### Desktop versions (Windows, OS X, Linux)

Copay uses NW.js (also know as node-webkit) for its desktop version. NW.js is an app runtime based on `Chromium` and `node.js`.

- Install NW.js on your system from [nwjs.io](http://nwjs.io/)
- Run `grunt desktop`

### General

Payblk implements a multisig wallet using [p2sh](https://en.bitcoin.it/wiki/Pay_to_script_hash) addresses.  It supports multiple wallets, each with its own configuration, such as 3-of-5 (3 required signatures from 5 participant peers) or 2-of-3.  To create a multisig wallet shared between multiple participants, Payblk requires the extended public keys of all the wallet participants.  Those public keys are then incorporated into the wallet configuration and combined to generate a payment address where funds can be sent into the wallet.  Conversely, each participant manages their own private key and that private key is never transmitted anywhere.

To unlock a payment and spend the wallet's funds, a quorum of participant signatures must be collected and assembled in the transaction.  The funds cannot be spent without at least the minimum number of signatures required by the wallet configuration (2-of-3, 3-of-5, 6-of-6, etc.).  Once a transaction proposal is created, the proposal is distributed among the wallet participants for each to sign the transaction locally.  Finally, when the transaction is signed, the last signing participant will broadcast the transaction to the Bitcoin network.

Copay also implements [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) to generate new addresses for peers.  The public key that each participant contributes to the wallet is a BIP32 extended public key.  As additional public keys are needed for wallet operations (to produce new addresses to receive payments into the wallet, for example) new public keys can be derived from the participants' original extended public keys.  Once again, it's important to stress that each participant keeps their own private keys locally - private keys are not shared - and are used to sign transaction proposals to make payments from the shared wallet.

For more information regarding how addresses are generated using this procedure, see: [Structure for Deterministic P2SH Multisignature Wallets](https://github.com/bitcoin/bips/blob/master/bip-0045.mediawiki).


## Wallet Export Format

Payblk encrypts the backup with the [Stanford JS Crypto Library](http://bitwiseshiftleft.github.io/sjcl/).  To extract the private key of your wallet you can use https://bitwiseshiftleft.github.io/sjcl/demo/, copy the backup to 'ciphertext' and enter your password.  The resulting JSON will have a key named: `xPrivKey`, that is the extended private key of your wallet.  That information is enough to sign any transaction from your wallet, so be careful when handling it!

The backup also contains the key `publicKeyRing` that holds the extended public keys of the Payblkrs.
Depending on the key `derivationStrategy`, addresses are derived using
[BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) or [BIP45](https://github.com/bitcoin/bips/blob/master/bip-0045.mediawiki). Wallets created in Copay v1.2 and forward always use BIP44, all previous wallets use BIP45. Also note that since Payblk version v1.2, non-multisig wallets use address types Pay-to-PublicKeyHash (P2PKH) while multisig wallets still use Pay-to-ScriptHash (P2SH) (key `addressType` at the backup):

| Copay Version  | Wallet Type   | Derivation Strategy   | Address Type  |
|---|---|---|---|---|
|  <1.2  | All  |  BIP45 | P2SH   |
|  >=1.2 | Non-multisig  | BIP44  | P2PKH   |
| >=1.2  | Multisig  |  BIP44 |  P2SH   |
| >=1.5  | Multisig Hardware wallets  |  BIP44 (root m/48') |  P2SH   |

Using a tool like [Bitcore PlayGround](http://bitcore.io/playground) all wallet addresses can be generated. (TIP: Use the `Address` section for P2PKH address type wallets and `Multisig Address` for P2SH address type wallets). For multisig addresses, the required number of signatures (key `m` on the export) is also needed to recreate the addresses.

BIP45 note: All addresses generated at BWS with BIP45 use the 'shared cosigner index' (2147483647) so Copay address indexes look like: `m/45'/2147483647/0/x` for main addresses and `m/45'/2147483647/1/y` for change addresses.

## Bitcore Wallet Service

Payblk depends on [Blackcore Wallet Service](https://github.com/janko33bd/bitcore-wallet-service/blob/blackore-wallet-service/) (BWS) for blockchain information, networking and Payblker synchronization.  A BWS instance can be setup and operational within minutes or you can use a public instance like `https://node.blackcoin.io/bws`.  Switching between BWS instances is very simple and can be done with a click from within Payblk.


## License

Copay is released under the MIT License.  Please refer to the [LICENSE](https://github.com/bitpay/copay/blob/master/LICENSE) file that accompanies this project for more information including complete terms and conditions.
