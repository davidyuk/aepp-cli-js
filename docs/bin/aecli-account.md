





  

```js
#!/usr/bin/env node

```







# æternity CLI `account` file

This script initialize all `account` commands


  

```js
/*
 * ISC License (ISC)
 * Copyright (c) 2018 aeternity developers
 *
 *  Permission to use, copy, modify, and/or distribute this software for any
 *  purpose with or without fee is hereby granted, provided that the above
 *  copyright notice and this permission notice appear in all copies.
 *
 *  THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH
 *  REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY
 *  AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT,
 *  INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM
 *  LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR
 *  OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
 *  PERFORMANCE OF THIS SOFTWARE.
 */

```







We'll use `commander` for parsing options

Also we need `esm` package to handle `ES imports`


  

```js
const program = require('commander')

require = require('esm')(module/*, options*/) //use to handle es6 import/export
const utils = require('./utils/index')
const { Account } = require('./commands')


```







## Initialize `options`


  

```js
program
  .option('-u, --url [hostname]', 'Node to connect to', utils.constant.NODE_URL)
  .option('-U, --internalUrl [internal]', 'Node to connect to(internal)', utils.constant.NODE_INTERNAL_URL)
  .option('-P, --password [password]', 'Wallet Password')
  .option('-f --force', 'Ignore epoch version compatibility check')
  .option('--json', 'Print result in json format')


```







## Initialize `spend` command

You can use this command to send tokens to another account

Example: `aecli account spend ./myWalletKeyFile ak_1241rioefwj23f2wfdsfsdsdfsasdf 100 --password testpassword`

Example: `aecli account spend ./myWalletKeyFile aensAccountName.chain 100 --password testpassword`

You can set transaction `ttl(Time to leave)`. If not set use default.

Example: `aecli account spend ./myWalletKeyFile ak_1241rioefwj23f2wfdsfsdsdfsasdf 100 --password testpassword --ttl 20` --> this tx will leave for 20 blocks


  

```js
program
  .command('spend <wallet_path> <receiverIdOrName> <amount>')
  .option('--networkId [networkId]', 'Network id (default: ae_mainnet)')
  .option('--payload [payload]', 'Transaction payload.', '')
  .option('-F, --fee [fee]', 'Spend transaction fee.')
  .option('-T, --ttl [ttl]', 'Validity of the spend transaction in number of blocks (default forever)', utils.constant.TX_TTL)
  .option('-N, --nonce [nonce]', 'Override the nonce that the transaction is going to be sent with')
  .option('-D, --denomination [denomination]', 'Denomination of amount', utils.constant.DENOMINATION)
  .action(async (walletPath, receiverIdOrName, amount, ...arguments) => await Account.spend(walletPath, receiverIdOrName, amount, utils.cli.getCmdFromArguments(arguments)))



```







## Initialize `transfer` command

You can use this command to send % of balance to another account

Example: `aecli account transfer ./myWalletKeyFile ak_1241rioefwj23f2wfdsfsdsdfsasdf 0.5 --password testpassword`

You can set transaction `ttl(Time to leave)`. If not set use default.

Example: `aecli account transfer ./myWalletKeyFile ak_1241rioefwj23f2wfdsfsdsdfsasdf 0.5 --password testpassword --ttl 20` --> this tx will leave for 20 blocks


  

```js
program
  .command('transfer <wallet_path> <receiver> <percentage>')
  .option('--excludeFee', 'Exclude fee from amount')
  .option('--networkId [networkId]', 'Network id (default: ae_mainnet)')
  .option('--payload [payload]', 'Transaction payload.', '')
  .option('-F, --fee [fee]', 'Spend transaction fee.')
  .option('-T, --ttl [ttl]', 'Validity of the spend transaction in number of blocks (default forever)', utils.constant.TX_TTL)
  .option('-N, --nonce [nonce]', 'Override the nonce that the transaction is going to be sent with')
  .option('-D, --denomination [denomination]', 'Denomination of amount', utils.constant.DENOMINATION)
  .action(async (walletPath, receiver, percentage, ...arguments) => await Account.transferFunds(walletPath, receiver, percentage, utils.cli.getCmdFromArguments(arguments)))



```







## Initialize `sign` command

You can use this command to sign your transaction's

Example: `aecli account sign ./myWalletKeyFile tx_1241rioefwj23f2wfdsfsdsdfsasdf --password testpassword`


  

```js
program
  .command('sign <wallet_path> <tx>')
  .option('--networkId [networkId]', 'Network id (default: ae_mainnet)')
  .description('Create a transaction to another wallet')
  .action(async (walletPath, tx, ...arguments) => await Account.sign(walletPath, tx, utils.cli.getCmdFromArguments(arguments)))



```







## Initialize `sign-message` command

You can use this command to sign message

Example: `aecli account sign-message ./myWalletKeyFile Hello --password testpassword`


  

```js
program
  .command('sign-message <wallet_path> [data...]')
  .option('--filePath [path]', 'Specify the path to the file for signing(ignore command message argument and use file instead)')
  .description('Create a transaction to another wallet')
  .action(async (walletPath, data, ...arguments) => await Account.signMessage(walletPath, data, utils.cli.getCmdFromArguments(arguments)))



```







## Initialize `verify-message` command

You can use this command to sign message

Example: `aecli account verify-message ./myWalletKeyFile asd1dasfadfsdasdasdasHexSig... Hello --password testpassword`


  

```js
program
  .command('verify-message <wallet_path> <hexSignature> [data...]')
  .option('--filePath [path]', 'Specify the path to the file(ignore comm and message argument and use file instead)')
  .description('Create a transaction to another wallet')
  .action(async (walletPath, hexSignature, data, ...arguments) => await Account.verifyMessage(walletPath, hexSignature, data, utils.cli.getCmdFromArguments(arguments)))



```







## Initialize `balance` command

You can use this command to retrieve balance of account

Example: `aecli account balance ./myWalletKeyFile --password testpassword`


  

```js
program
  .command('balance <wallet_path>')
  .option('--height [height]', 'Specific block height')
  .option('--hash [hash]', 'Specific block hash')
  .description('Get wallet balance')
  .action(async (walletPath, ...arguments) => await Account.getBalance(walletPath, utils.cli.getCmdFromArguments(arguments)))


```







## Initialize `address` command

You can use this command to retrieve get your public and private key

Example: `aecli account address ./myWalletKeyFile --password testpassword` --> show only public key

Example: `aecli account address ./myWalletKeyFile --password testpassword --privateKey` --> show  public key and private key


  

```js
program
  .command('address <wallet_path>')
  .option('--privateKey', 'Print private key')
  .option('--forcePrompt', 'Force prompting')
  .description('Get wallet address')
  .action(async (walletPath, ...arguments) => await Account.getAddress(walletPath, utils.cli.getCmdFromArguments(arguments)))


```







## Initialize `create` command

You can use this command to generate `keypair` and encrypt it by password.
This command create `ethereum like keyfile`.

You can use `--output ./keys` to set directory to save you key.

Example: `aecli account create myWalletName --password testpassword`

Example: `aecli account create myWalletName --password testpassword --output ./mykeys` --> create `key-file` in `mykeys` directory


  

```js
program
  .command('create <name>')
  .option('-O, --output [output]', 'Output directory', '.')
  .option('--overwrite', 'Overwrite if exist')
  .description('Create a secure wallet')
  .action(async (name, ...arguments) => await Account.createSecureWallet(name, utils.cli.getCmdFromArguments(arguments)))


```







## Initialize `save` command

You can use this command to generate `keypair` from `private-key` and encrypt it by password.
This command create `ethereum like keyfile`.

You can use `--output ./keys` to set directory to save you key

Example: `aecli account save myWalletName 1902855723940510273412074210842018342148234  --password testpassword`

Example: `aecli account save myWalletName 1902855723940510273412074210842018342148234 --password testpassword --output ./mykeys` --> create `key-file` in `mykeys` directory


  

```js
program
  .command('save <name> <privkey>')
  .option('-O, --output [output]', 'Output directory', '.')
  .option('--overwrite', 'Overwrite if exist')
  .description('Save a private keys string to a password protected file wallet')
  .action(async (name, priv, ...arguments) => await Account.createSecureWalletByPrivKey(name, priv, utils.cli.getCmdFromArguments(arguments)))


```







## Initialize `nonce` command

You can use this command to get `account nonce`.

You can use `--output ./keys` to set directory to save you key

Example: `aecli account nonce myWalletName --password testpassword


  

```js
program
  .command('nonce <wallet_path>')
  .description('Get account nonce')
  .action(async (walletPath, ...arguments) => await Account.getAccountNonce(walletPath, utils.cli.getCmdFromArguments(arguments)))


```







## Initialize `generateKeyPairs` command

You can use this command to generate KeyPair's.

Example: `aecli account generate 10 --force


  

```js
program
  .command('generate <count>')
  .option('--forcePrompt', 'Force prompting')
  .description('Generate keyPairs')
  .action(async (count, ...arguments) => await Account.generateKeyPairs(count, utils.cli.getCmdFromArguments(arguments)))



```







Handle unknown command's


  

```js
program.on('command:*', () => utils.errors.unknownCommandHandler(program)())


```







Parse arguments or show `help` if argument's is empty


  

```js
program.parse(process.argv)
if (program.args.length === 0) program.help()


```




