# ContractKit v1.0으로 마이그레이션하기

cLabs는 최근 [ContractKit 버전 1.0.0](https://medium.com/celoorg/contractkit-1-0-0-9c0412462d45)을 배포했습니다. 해당 버전에서는 원래 ContractKit 패키지가 모두 Celo SDK를 구성하는 여러 개별 패키지로 분할되었습니다. 이 문서에서는 주요 차이점을 설명하고 업데이트된 SDK를 사용하는 방법을 보여 줍니다.

이전 버전의 ContractKit(1.0.0 이하 버전)를 사용하는 경우 해당 버전을 계속 사용할 수 있으며 업그레이드할 때 뒤의 내용과 같이 변경하기만 하면 됩니다.

새 버전을 사용할 때의 주요 이점은 다음과 같습니다:
 - 번들 사이즈 감소
 - 더 나은 타입스크립트 지원
 - 다른 라이브러리를 보다 쉽게 사용할 수 있도록 하여 유지 관리 개선

## ContractKit 패키지

ContractKit 이제 [패키지 세트](https://github.com/celo-org/celo-monorepo/tree/master/packages/sdk)입니다.

### 주요 패키지

 - `Connect` 는 체인 노드와 통신하는 방법을 다룹니다. `web3` 라이브러리를 래핑하고 자체 `rpcCaller` 클래스가 있어 노드에 사용자 지정 호출을 수행합니다. 이 계층은 Celo가 어떤 매개변수를 어떻게 추가하고 노드에 연결하고, 메시지를 작성하고, 메시지를 작성하고, 보내고, 응답을 처리하는지 파악하는 계층입니다.
 - `ContractKit`는 이전 버전의 ContractKit의 축소된 부분집합입니다. 이 계층은 [core contracts](contracts-wrappers-registry.md)를 로드하고 사용하는 계층입니다. 내부적으로는 위에서 설명한 `connect` 패키지를 사용합니다. ABI에서 생성된 컨트랙트, 래퍼 및 클레임을 만드는 로직을 포함합니다.

### 보완 패키지

 - `Explorer`는 `contractkit`와 `connect`에 따라 달라집니다. 새 블록 및 로그 정보를 쉽게 수신할 수 있는 몇 가지 유틸리티 기능을 제공합니다.
 - `Governance`는 `contractkit`와 `explorer`에 따라 달라집니다. Celo Governance Proposes(CGP)를 읽고 상호 작용하는 기능을 제공합니다.
 - `Identity`는 전화 번호를 기반으로 하는 Celo의 경량 식별 계층인 [ODIS](odis.md)과의 상호 작용을 단순화합니다.
 - `Network-utils`는 제네시스 블록 및 정적 노드 정보를 얻기 위한 유틸리티를 제공합니다.
 - `Transactions-uri`를 사용하여 Celo 트랜잭션 URI와 QR 코드를 쉽게 생성할 수 있습니다.

### 지갑 및 지갑 유틸리티 패키지

 - `Wallet-hsm-azure`은 RemoteWallet의 Azure Key Vault 구현입니다.
 - `Wallet-hsm-aws`를 사용하면 AWS KMS 기반 클라우드 HSM 지갑과 쉽게 상호 작용할 수 있습니다.
 - `Wallet-ledger`는 레저 하드웨어 지갑과 상호 작용하기 위한 유틸리티를 제공합니다.
 - `Wallet-local`은 개인키 문자열을 가져와 로컬에서 지갑을 관리하기 위한 유틸리티를 제공합니다.
 - `Wallet-rpc`는 RPC를 통해 지갑 기능을 수행하기 위한 유틸리티를 제공합니다.
 - `Wallet-base`는 Celo 지갑을 만드는 기본 유틸리티를 제공합니다.
 - `Wallet-hsm`은 HSM을 사용하기 위한 시그니처 유틸리티를 제공합니다.
 - `Wallet-remote`는 원격 지갑과 상호 작용하기 위한 유틸리티를 제공합니다. 보안 원격 서버의 지갑과 상호 작용할 때 유용합니다.

## 패키지 가져오기

Importing the packages is slightly different now that many packages are separate from the main `ContractKit` package. You will have to explicitly import these packages instead of just importing all of them with `ContractKit`.
현재 많은 패키지가 메인 `ContractKit` 패키지와 분리되어 있기 때문에 패키지를 가져오는 것이 조금씩 다릅니다. 이 패키지들은 단순히 'ContractKit'로 모두 가져오는 대신 명시적으로 가져와야 합니다.

예를 들어:

```javascript
// Previously this would work to import the block-explorer
import { newBlockExplorer } from '@celo/contractkit/lib/explorer/block-explorer'

// With ContractKit v1.x.y, import the block-explorer explicitly
import { newBlockExplorer } from '@celo/explorer/lib/block-explorer'
```

## 네트워크에 연결하기 

### ContractKit의 구버전:

```javascript
// version ^0.4.0 
const ContractKit = require('@celo/contractkit')

// Older versions of ContractKit create a new Web3 instance internally 
const kit = ContractKit.newKit('https://forno.celo.org')
```

### 버전 1.x.y

```javascript
// Since ContractKit no longer instantiates web3, you'll need to explicitly require it 
const Web3 = require('web3') 
const web3 = new Web3('https://forno.celo.org') 

// Require ContractKit and newKitFromWeb3 
const { ContractKit, newKitFromWeb3 } = require('@celo/contractkit') 
let contractKit = newKitFromWeb3(web3)
```
## Web3 함수에 접근하기

`connection` 모듈을 사용하여 `web3`함수에 접근할 수 있습니다.

```javascript
// version ^0.4.0 
let amount = kit.web3.utils.fromWei("1000000", "ether")
 
// version 1.x.y
let amount = kit.connection.web3.utils.fromWei("1000000", "ether")
```
## 이전 버전과의 호환

[이 ContractKit 함수](https://github.com/celo-org/celo-monorepo/blob/a7579fc9bdc0c1b4ce1d9fec702938accf82be2a/packages/sdk/contractkit/src/kit.ts#L278)들은 `kit`로 직접 접근해도 여전히 작동하지만 향후 호환되지 않는 문제를 피하려면 `connection`을 통해 사용하는 것이 좋습니다.

```
// This still works
kit.addAccount

// recommended:
kit.connection.addAccount
```

## `Connection` 패키지

`connection` 패키지 업데이트는 몇 가지 일반적인 Web3 기능의 구현이 포함되어 있습니다. 다음은 몇 가지 예시입니다:

 - `kit.web3.eth.isSyncing` --> `kit.connection.isSyncing`
 - `kit.web3.eth.getBlock` --> `kit.connection.getBlock`
 - `kit.web3.eth.getBlockNumber` --> `kit.connection.getBlockNumber`
 - `kit.web3.eth.sign` --> `kit.connection.sign`
 - `kit.isListening` --> `kit.connection.isListening`
 - `kit.addAccount` --> `kit.connection.addAccount`
