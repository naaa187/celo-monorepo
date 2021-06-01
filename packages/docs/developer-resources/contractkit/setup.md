# 시작하기

## 설치 및 시스템 요구사항

다음을 실행하여 설치를 진행하세요.
```bash
npm install @celo/contractkit
// or
yarn add @celo/contractkit
```

Node.js v12.x.이 필요합니다.

## 키트를 초기화하기

ContractKit로 작업을 시작하려면, `kit` 인스턴스와 연결할 valid net이 필요합니다. 본 예제에서는 `alfajores`를 사용합니다. (더 많은 정보는 [링크](../../getting-started/alfajores-testnet.md)에서 확인할 수 있습니다.)

```ts
import { newKit } from '@celo/contractkit'

const kit = newKit('https://alfajores-forno.celo-testnet.org')
```

다른 연결 유형과 네트워크 엔드포인트에 대한 자세한 사항은 [Forno 관련 문서](../forno/README.md)에서 확인하세요.

## 자신의 노드로 키트를 초기화하기

만일 호스팅 중인 노드가 있다면 (노드를 실행하고 싶다면 [이 가이드](../../getting-started/running-a-full-node-in-mainnet.md)를 참고하세요.) ContractKit를 해당 노드에 연결할 수 있습니다.

```ts
import { newKit } from '@celo/contractkit'

// define localUrl and port with the ones of your node

const kit = newKit(`${localUrl}:${port}`)
```

`Web3`와 마찬가지로, `WebSockets`와 `RPC`, `IPC`를 통한 연결을 지원합니다.
마지막의 경우, `kit`를 **유효한** `IPC Provider`를 가지고 있는 `Web3`의 인스턴스로 초기화해야합니다.

```ts
import Web3 from 'web3'
import { newKitFromWeb3 } from '@celo/contractkit'

const web3Instance: Web3 = new Web3(new Web3.providers.IpcProvider('/Users/myuser/Library/CeloNode/geth.ipc', net))

const kit = newKitFromWeb3(web3Instance)
```

