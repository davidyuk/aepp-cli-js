





  

```js
#!/usr/bin/env node

```







# æternity CLI `transaction` file

This script initialize all `transaction` function


  

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

import { encodeBase58Check, salt, assertedType } from '@aeternity/aepp-sdk/es/utils/crypto'
import { commitmentHash } from '@aeternity/aepp-sdk/es/tx/builder/helpers'
import { TX_TYPE } from '@aeternity/aepp-sdk/es/tx/builder/schema'

import { initChain, initOfflineTxBuilder, initTxBuilder } from '../utils/cli'
import { handleApiError } from '../utils/errors'
import { print, printBuilderTransaction, printError, printUnderscored, printValidation } from '../utils/print'
import { validateName } from '../utils/helpers'
import { BUILD_ORACLE_TTL, ORACLE_VM_VERSION, DEFAULT_CONTRACT_PARAMS } from '../utils/constant'


```







## Build `spend` transaction


  

```js
async function spend (senderId, recipientId, amount, nonce, options) {
  let { ttl, json, fee, payload } = options
  ttl = parseInt(ttl)
  nonce = parseInt(nonce)
  try {

```







Initialize `Ae`


  

```js
    const txBuilder = initOfflineTxBuilder()

```







Build params


  

```js
    const params = {
      senderId,
      recipientId,
      amount,
      ttl,
      nonce,
      fee,
      payload
    }

```







calculate fee


  

```js
    fee = txBuilder.calculateFee(fee, TX_TYPE.spend, { params })

```







Build `spend` transaction


  

```js
    const tx = txBuilder.buildTx({ ...params, fee }, TX_TYPE.spend)

```







Print Result


  

```js
    if (json) print({ tx: tx.tx, params: tx.txObject })
    else printBuilderTransaction(tx, TX_TYPE.spend)
  } catch (e) {
    printError(e)
    process.exit(1)
  }
}


```







## Build `namePreClaim` transaction


  

```js
async function namePreClaim (accountId, domain, nonce, options) {
  let { ttl, json, fee } = options

  try {

```







Validate `name`(check if `name` end on `.test`)


  

```js
    validateName(domain)

```







Initialize `Ae`


  

```js
    const txBuilder = initOfflineTxBuilder()


```







Generate `salt` and `commitmentId` and build `name` hash


  

```js
    const _salt = salt()
    const commitmentId = await commitmentHash(domain, _salt)

    const params = {
      accountId,
      commitmentId,
      ttl,
      nonce
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.namePreClaim, { params })

```







Create `preclaim` transaction


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.namePreClaim)

    if (json) {
      print({ tx, txObject, salt: _salt })
    } else {
      printBuilderTransaction({ tx, txObject: { ...txObject, salt: _salt } }, TX_TYPE.namePreClaim)
    }
  } catch (e) {
    printError(e)
    process.exit(1)
  }
}


```







## Build `nameClaim` transaction


  

```js
async function nameClaim (accountId, nameSalt, domain, nonce, options) {
  let { ttl, json, fee } = options
  const nameHash = `nm_${encodeBase58Check(Buffer.from(domain))}`

  try {

```







Validate `name`(check if `name` end on `.test`)


  

```js
    validateName(domain)

```







Initialize `Ae`


  

```js
    const txBuilder = initOfflineTxBuilder()
    const params = {
      accountId,
      nameSalt,
      name: nameHash,
      ttl,
      nonce
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.nameClaim, { params })

```







Build `claim` transaction's


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.nameClaim)

    if (json) {
      print({ tx, txObject })
    } else {
      printBuilderTransaction({ tx, txObject }, TX_TYPE.nameClaim)
    }
  } catch (e) {
    printError(e)
    process.exit(1)
  }
}

function classify (s) {
  const keys = {
    ak: 'account_pubkey',
    ok: 'oracle_pubkey',
    ct: 'contract_pubkey',
  }

  if (!s.match(/^[a-z]{2}_.+/)) {
    throw Error('Not a valid hash')
  }

  const klass = s.substr(0, 2)
  if (klass in keys) {
    return keys[klass]
  } else {
    throw Error(`Unknown class ${klass}`)
  }
}


```







## Build `nameUpdate` transaction


  

```js
async function nameUpdate (accountId, nameId, nonce, pointers, options) {
  let { ttl, json, fee, nameTtl, clientTtl } = options
  try {

```







Initialize `Ae`


  

```js
    const txBuilder = initOfflineTxBuilder()

```







Create `update` transaction


  

```js
    pointers = pointers.map(id => Object.assign({}, { id, key: classify(id) }))
    console.log(pointers)
    const params = {
      nameId,
      accountId,
      nameTtl,
      pointers,
      clientTtl,
      ttl,
      nonce
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.nameUpdate, { params })

```







Build `claim` transaction's


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.nameUpdate)

    if (json) {
      print({ tx, txObject })
    } else {
      printBuilderTransaction({ tx, txObject }, TX_TYPE.nameUpdate)
    }
  } catch (e) {
    printError(e)
    process.exit(1)
  }
}


```







## Build `nameTransfer` transaction


  

```js
async function nameTransfer (accountId, recipientId, nameId, nonce, options) {
  let { ttl, json, fee } = options
  try {

```







Initialize `Ae`


  

```js
    const txBuilder = initOfflineTxBuilder()

```







Create `transfer` transaction


  

```js
    const params = {
      accountId,
      recipientId,
      nameId,
      ttl,
      nonce
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.nameTransfer, { params })

```







Build `claim` transaction's


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.nameTransfer)

    if (json) {
      print({ tx, txObject })
    } else {
      printBuilderTransaction({ tx, txObject }, TX_TYPE.nameTransfer)
    }
  } catch (e) {
    printError(e)
    process.exit(1)
  }
}


```







## Build `nameRevoke` transaction


  

```js
async function nameRevoke (accountId, nameId, nonce, options) {
  let { ttl, json, fee } = options
  try {

```







Initialize `Ae`


  

```js
    const txBuilder = initOfflineTxBuilder()

```







Create `transfer` transaction


  

```js
    const params = {
      accountId,
      nameId,
      ttl,
      nonce
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.nameRevoke, { params })

```







Build `claim` transaction's


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.nameRevoke)

    if (json) {
      print({ tx, txObject })
    } else {
      printBuilderTransaction({ tx, txObject }, TX_TYPE.nameRevoke)
    }
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}


```







## Build `contractDeploy` transaction


  

```js
async function contractDeploy (ownerId, contractByteCode, initCallData, nonce, options) {
  let { ttl, json, fee, gas, deposit = 0, amount = 0 } = options
  ttl = parseInt(ttl)
  nonce = parseInt(nonce)
  try {

```







Initialize `Ae`


  

```js
    const txBuilder = await initTxBuilder(options)

```







Build `deploy` transaction's


  

```js
    await handleApiError(async () => {

```







Create `contract-deploy` transaction


  

```js
      const { tx, contractId } = await txBuilder.contractCreateTx({
        ...DEFAULT_CONTRACT_PARAMS,
        code: contractByteCode,
        nonce,
        fee,
        ttl,
        gas,
        ownerId,
        callData: initCallData,
        amount,
        deposit
      })

      if (json) {
        print(JSON.stringify({ tx, contractId }))
      } else {
        printUnderscored('Unsigned Contract Deploy TX', tx)
        printUnderscored('Contract ID', contractId)
      }
    })
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}


```







## Build `contractCall` transaction


  

```js
async function contractCall (callerId, contractId, callData, nonce, options) {
  let { ttl, json, fee, gas } = options
  nonce = parseInt(nonce)
  try {

```







Build `call` transaction's


  

```js
    await handleApiError(async () => {

```







Initialize `Ae`


  

```js
      const txBuilder = await initTxBuilder(options)

```







Create `contract-call` transaction


  

```js
      const tx = await txBuilder.contractCallTx({
        ...DEFAULT_CONTRACT_PARAMS,
        callerId,
        nonce,
        ttl,
        fee,
        gas,
        callData,
        contractId
      })

      if (json)
        print(JSON.stringify({ tx }))
      else
        printUnderscored('Unsigned Contract Call TX', tx)
    })
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}


```







## Build `oracleRegister` transaction


  

```js
async function oracleRegister (accountId, queryFormat, responseFormat, nonce, options) {
  let { ttl, json, fee, queryFee, oracleTtl } = options
  queryFee = parseInt(queryFee)
  oracleTtl = BUILD_ORACLE_TTL(parseInt(oracleTtl))
  nonce = parseInt(nonce)

  try {
    const txBuilder = initOfflineTxBuilder()

```







Create `transfer` transaction


  

```js
    const params = {
      accountId,
      ttl,
      fee,
      nonce,
      oracleTtl,
      queryFee,
      queryFormat,
      responseFormat,
      vmVersion: ORACLE_VM_VERSION
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.oracleRegister, { params })

```







Build `claim` transaction's


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.oracleRegister)
    if (json)
      print({ tx, txObject })
    else
      printBuilderTransaction({ tx, txObject }, TX_TYPE.oracleRegister)
  } catch (e) {
    printError(e)
    process.exit(1)
  }
}


```







## Build `oraclePostQuery` transaction


  

```js
async function oraclePostQuery (senderId, oracleId, query, nonce, options) {
  let { ttl, json, fee, queryFee, queryTtl, responseTtl } = options
  queryFee = parseInt(queryFee)
  queryTtl = BUILD_ORACLE_TTL(parseInt(queryTtl))
  responseTtl = BUILD_ORACLE_TTL(parseInt(responseTtl))
  nonce = parseInt(nonce)

  try {
    const txBuilder = initOfflineTxBuilder()

```







Create `transfer` transaction


  

```js
    const params = {
      senderId,
      ttl,
      fee,
      nonce,
      oracleId,
      query,
      queryFee,
      queryTtl,
      responseTtl
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.oracleQuery, { params })

```







Build `claim` transaction's


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.oracleQuery)

    if (json)
      print({ tx, txObject })
    else
      printBuilderTransaction({ tx, txObject }, TX_TYPE.oracleQuery)
  } catch (e) {
    printError(e)
    process.exit(1)
  }
}


```







## Build `oracleExtend` transaction


  

```js
async function oracleExtend (callerId, oracleId, oracleTtl, nonce, options) {
  let { ttl, json, fee } = options
  oracleTtl = BUILD_ORACLE_TTL(parseInt(oracleTtl))
  nonce = parseInt(nonce)

  try {
    const txBuilder = initOfflineTxBuilder()

```







Create `transfer` transaction


  

```js
    const params = {
      callerId,
      oracleId,
      oracleTtl,
      fee,
      nonce,
      ttl
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.oracleExtend, { params })

```







Build `claim` transaction's


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.oracleExtend)

    if (json)
      print({ tx, txObject })
    else
      printBuilderTransaction({ tx, txObject }, TX_TYPE.oracleExtend)
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}


```







## Build `oracleRespond` transaction


  

```js
async function oracleRespond (callerId, oracleId, queryId, response, nonce, options) {
  let { ttl, json, fee, responseTtl } = options
  responseTtl = BUILD_ORACLE_TTL(parseInt(responseTtl))
  nonce = parseInt(nonce)

  try {
    const txBuilder = initOfflineTxBuilder()

```







Create `transfer` transaction


  

```js
    const params = {
      oracleId,
      responseTtl,
      callerId,
      queryId,
      response,
      nonce,
      fee,
      ttl
    }
    fee = txBuilder.calculateFee(fee, TX_TYPE.oracleResponse, { params })

```







Build `claim` transaction's


  

```js
    const { tx, txObject } = txBuilder.buildTx({ ...params, fee }, TX_TYPE.oracleResponse)

    if (json)
      print({ tx, txObject })
    else
      printBuilderTransaction({ tx, txObject }, TX_TYPE.oracleResponse)
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}


```







## Verify 'transaction'


  

```js
async function verify (txHash, options) {
  let { json, networkId } = options
  try {

```







Validate input


  

```js
    if (!assertedType(txHash, 'tx')) throw new Error('Invalid transaction, must be lik \'tx_23didf2+f3sd...\'')

```







Initialize `Ae`


  

```js
    const client = await initChain(options)

```







Call `getStatus` API and print it


  

```js
    await handleApiError(async () => {
      const { validation, tx, signatures = [], txType: type } = await client.unpackAndVerify(txHash, { networkId })
      if (json) {
        print({ validation, tx: tx, signatures, type })
        process.exit(1)
      }
      printValidation({ validation, tx: { ...tx, signatures: signatures.map(el => el.hash) }, txType: type })
      if (!validation.length) print(' ✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓ TX VALID ✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓')
    })
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}

export const Transaction = {
  spend,
  namePreClaim,
  nameClaim,
  nameUpdate,
  nameRevoke,
  nameTransfer,
  contractDeploy,
  contractCall,
  oracleRegister,
  oraclePostQuery,
  oracleExtend,
  oracleRespond,
  verify
}


```



