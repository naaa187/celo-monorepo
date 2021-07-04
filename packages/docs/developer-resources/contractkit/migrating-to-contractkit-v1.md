# ContractKit v1.0으로 마이그레이션하기

cLabs는 최근 [ContractKit 버전 1.0.0](https://medium.com/celoorg/contractkit-1-0-0-9c0412462d45)을 배포했습니다. 해당 버전에서는 원래 ContractKit 패키지가 모두 Celo SDK를 구성하는 여러 개별 패키지로 분할되었습니다. 이 문서에서는 주요 차이점을 설명하고 업데이트된 SDK를 사용하는 방법을 보여 줍니다.

이전 버전의 ContractKit(1.0.0 이하 버전)를 사용하는 경우 해당 버전을 계속 사용할 수 있으며 업그레이드할 때 뒤의 내용과 같이 변경하기만 하면 됩니다.

새 버전을 사용할 때의 주요 이점은 다음과 같습니다:
 - 번들 사이즈 감소
 - 더 나은 타입스크립트 지원
 - 다른 라이브러리를 보다 쉽게 사용할 수 있도록 하여 유지 관리 개선
 - 
## ContractKit 패키지

ContractKit 이제 [패키지 세트](https://github.com/celo-org/celo-monorepo/tree/master/packages/sdk)입니다.

### 주요 패키지

 - `Connect` 는 체인 노드와 통신하는 방법을 다룹니다. `web3` 라이브러리를 래핑하고 자체 `rpcCaller` 클래스가 있어 노드에 사용자 지정 호출을 수행합니다. 이 계층은 Celo가 어떤 매개변수를 어떻게 추가하고 노드에 연결하고, 메시지를 작성하고, 메시지를 작성하고, 보내고, 응답을 처리하는지 파악하는 계층입니다.
 - `ContractKit`는 이전 버전의 ContractKit의 축소된 부분집합입니다. 이 계층은 [core contracts](contracts-wrappers-registry.md)를 로드하고 사용하는 계층입니다. 내부적으로는 위에서 설명한 `connect` 패키지를 사용합니다. ABI에서 생성된 컨트랙트, 래퍼 및 클레임을 만드는 로직을 포함합니다.

### 보완 패키지

 - `Explorer`는 `contractkit`와 `connect`에 따라 달라집니다. It provides some utility functions that make it easy to listen for new block and log information.
 - `Governance` depends on `contractkit` and `explorer`. It provides functions to read and interact with Celo Governance Proposals (CGPs).
 - `Identity` simplifies interacting with [ODIS](odis.md), Celo’s lightweight identity layer based on phone numbers.
 - `Network-utils` provides utilities for getting genesis block and static node information.
 - `Transactions-uri` makes it easy to generate Celo transaction URIs and QR codes.

### Wallets and Wallet Utility packages

 - `Wallet-hsm-azure` is a Azure Key Vault implementation of a RemoteWallet.
 - `Wallet-hsm-aws` allows you to easily interact with a cloud HSM wallet built on AWS KMS.
 - `Wallet-ledger` provides utilities for interacting with a Ledger hardware wallet.
 - `Wallet-local`provides utilities for locally managing wallet by importing a private key string.
 - `Wallet-rpc` provides utilities for performing wallet functions via RPC.
 - `Wallet-base` provides base utilities for creating Celo wallets.
 - `Wallet-hsm` provides signature utilities for using HSMs.
 - `Wallet-remote` provides utilities for interacting with remote wallets. This is useful for interacting with wallets on secure remote servers.

## Importing packages

Importing the packages is slightly different now that many packages are separate from the main `ContractKit` package. You will have to explicitly import these packages instead of just importing all of them with `ContractKit`.

For example:

```javascript
// Previously this would work to import the block-explorer
import { newBlockExplorer } from '@celo/contractkit/lib/explorer/block-explorer'

// With ContractKit v1.x.y, import the block-explorer explicitly
import { newBlockExplorer } from '@celo/explorer/lib/block-explorer'
```

## Connecting to the network

### Older versions of ContractKit:

```javascript
// version ^0.4.0 
const ContractKit = require('@celo/contractkit')

// Older versions of ContractKit create a new Web3 instance internally 
const kit = ContractKit.newKit('https://forno.celo.org')
```

### Version 1.x.y

```javascript
// Since ContractKit no longer instantiates web3, you'll need to explicitly require it 
const Web3 = require('web3') 
const web3 = new Web3('https://forno.celo.org') 

// Require ContractKit and newKitFromWeb3 
const { ContractKit, newKitFromWeb3 } = require('@celo/contractkit') 
let contractKit = newKitFromWeb3(web3)
```
## Accessing Web3 functions

You can access `web3` functions through the `connection` module.

```javascript
// version ^0.4.0 
let amount = kit.web3.utils.fromWei("1000000", "ether")
 
// version 1.x.y
let amount = kit.connection.web3.utils.fromWei("1000000", "ether")
```
## Backward Compatibility

[These ContractKit functions](https://github.com/celo-org/celo-monorepo/blob/a7579fc9bdc0c1b4ce1d9fec702938accf82be2a/packages/sdk/contractkit/src/kit.ts#L278) will still work when accessed directly from `kit`, but it is advised to consume it via `connection` to avoid future deprecation.

```
// This still works
kit.addAccount

// recommended:
kit.connection.addAccount
```

## `Connection` package

The `connection` package update includes implementations of some common web3 functions. Here are a few examples:

 - `kit.web3.eth.isSyncing` --> `kit.connection.isSyncing`
 - `kit.web3.eth.getBlock` --> `kit.connection.getBlock`
 - `kit.web3.eth.getBlockNumber` --> `kit.connection.getBlockNumber`
 - `kit.web3.eth.sign` --> `kit.connection.sign`
 - `kit.isListening` --> `kit.connection.isListening`
 - `kit.addAccount` --> `kit.connection.addAccount`
