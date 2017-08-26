# BigchainDB-ORM

> A CRAB-based ORM for BigchainDB.

CRAB is the CRUD model in databases applied to blockchains:

| Database     | Blockchain  |
| ------------ | ----------- |
| **C**reate   | **C**reate  |
| **R**eceive  | **R**eceive |
| **U**pdate   | **A**ppend  |
| **D**elete   | **B**urn    |

## Table of Contents

- [Setup](#setup)
- [Usage](#usage)
- [Examples](#examples)
    - [Create an asset](#example-create-an-asset)
    - [Receive assets](#example-receive-assets)
    - [Append a transaction](#example-append-a-transaction)
    - [Burn an asset](#example-burn-an-asset)
- [License](#license)

## Setup

```bash
$ npm install bigchaindb-orm
```

## Usage

```javascript
// import bigchaindb-orm
import * as Orm from 'bigchaindb-orm'
// connect to BigchainDB
const bdbOrm = new Orm(
    "https://test.ipdb.io/api/v1/",
    {
        app_id: "Get one from developers.ipdb.io",
        app_key: "Same as app_id"
    }
)
// define our models and assets
bdbOrm.define("myModel", "https://schema.org/v1/myModel")
// create a public and private key for Alice
const aliceKeypair = new driver.Ed25519Keypair()
```

## Examples

All examples need bdbOrm initialized as described in usage

### Example: Create an asset

```javascript
// from the defined models in our bdbOrm we create an asset with Alice as owner
bdbOrm.myModel.create({
        keypair: aliceKeypair,
        metadata: { key: 'metadataValue' }
    })
    .then(asset => {
        /* 
            asset is an object with all our data and functions
            asset.id equals the id of the asset
            asset.metadata is metadata of the last (unspent) transaction
            asset.transactionList gives the full transaction history
        */
        console.log(asset.id)
    })
```

### Example: Receive assets

```javascript
// get all objects with retrieve()
// or get a specific object with retrieve(object.id)
bdbOrm.myModel.retrieve()
    .then(assets => {
        // assets is an array of myModel
        console.log(assets.map(asset => asset.id))
    })
```

### Example: Append a transaction
> "Update" (Database) => "Append" (Blockchain)

```javascript
// create an asset with Alice as owner
bdbOrm.myModel.create({
        keypair: aliceKeypair,
        metadata: { key: 'metadataValue' }
    })
    .then(asset => {
        // lets append update the metadata of our asset
        // since we use a blockchain, we can only append
        return asset.append({
            toKeypair: aliceKeypair.publicKey,
            keypair: aliceKeypair,
            metadata: { key: 'updatedValue' }
        })
    })   
    .then(updatedAsset => {
        // updatedAsset contains the last (unspent) state
        // of our asset so any actions
        // need to be done to updatedAsset
        console.log(updatedAsset.metadata)
    })
```

### Example: Burn an asset
> "Delete" (Database) => "Burn" (Blockchain)

```javascript
// create an asset with Alice as owner
bdbOrm.myModel.create({
        keypair: aliceKeypair,
        metadata: { key: 'metadataValue' }
    })
    .then(asset => {
        // lets burn the asset by assigning to a random keypair
        // since will not store the private key it's infeasible to redeem the asset
        return asset.burn({
            keypair: aliceKeypair
        })
    })
    .then((burnedAsset)=>{
        // asset is now tagged as "burned"
        console.log(burnedAsset.metadata)
    })
```

## License

```
Copyright 2017 BigchainDB GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
