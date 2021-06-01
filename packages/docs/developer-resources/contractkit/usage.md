# ContractKit 사용법

다음의 내용은 이미 노드에 연결된 상태라는 가정 하에 `ContractKit`로 할 수 있는 몇가지 예시들 입니다. 만일 연결을 완료하지 않았다면, [이곳의 복습 자료](../walkthroughs/hellocontracts.md#deploy-to-alfajores)를 봐주세요.

## 기본 Tx 옵션 설정하기

`kit`는 기본 트랜잭션 옵션을 설정하게 합니다.

```ts
import { CeloContract } from '@celo/contractkit'

let accounts = await kit.web3.eth.getAccounts()
kit.defaultAccount = accounts[0]
// paid gas in cUSD
await kit.setFeeCurrency(CeloContract.StableToken)
```

## 잔고 총액 얻기

`kit`에서 아래의 메서드를 사용하면 해당 주소의 CELO, locked CELO, cUSD와 총 잔액을 반환합니다.

```ts
let totalBalance = await kit.getTotalBalance(myAddress)
```

## 컨트랙트 배포하기

이미 기본 계정으로 컨트랙트가 배포되어 있습니다. 간단하게 `to:` 필드가 없는 트랜잭션을 보내세요. 사용자 정의 트랜잭션을 보내는 방법은 아래 항목을 참고하세요. 

[Alfajores 블록 탐색기](https://alfajores-blockscout.celo-testnet.org/)에서 배포를 확인할 수 있습니다. 영수증이 나오길 기다린 후 로그로 거래 내역을 받아오세요.

```ts
let bytecode = '0x608060405234...' // compiled Solidity deployment bytecode

let tx = await kit.sendTransaction({
    data: bytecode
})

let receipt = tx.waitReceipt()
console.log(receipt)
```

## 사용자 정의 트랜잭션 전송하기

Celo 트랜잭션 객체는 Ethereum의 객채와 다릅니다. 다음과 같이 세 가지 새로운 필드가 있습니다.

- `feeCurrency` (가스와 게이트웨이 요금을 지불할 때 사용할 ERC20 컨트랙트의 주소)
- `gatewayFeeRecipient` (라이트 클라이언트의 전체 트랜잭션의 코인베이스 주소)
- `gatewayFee` (요금 통화로 표기된 게이트웨이 요금 수취인에게 지급되는 값)

이것은 `web3.eth.sendTransaction`이나 `myContract.methods.transfer().send()`을 사용하는 것을 **피해야 한다**는 것을 뜻합니다.

대신, `contractkit`는 두 시나리오 모두에서 트랜잭션을 전송하는 유틸리티 메서드를 제공합니다. **컨트랙트 래퍼를 사용하면 이것을 사용할 필요가 없습니다**

원 트랜잭션의 경우:

```ts
const tx = kit.sendTransaction({
  from: myAddress,
  to: someAddress,
  value: oneGold,
})
const hash = await tx.getHash()
const receipt = await tx.waitReceipt()
```

a web3 컨트랙트 객체와 상호작용할 때:

```ts
const goldtoken = await kit._web3Contracts.getGoldToken()
const oneGold = kit.web3.utils.toWei('1', 'ether')

const txo = await goldtoken.methods.transfer(someAddress, oneGold)
const tx = await kit.sendTransactionObject(txo, { from: myAddress })
const hash = await tx.getHash()
const receipt = await tx.waitReceipt()
```

## 사용자 정의 컨트랙트와 상호작용하기

컨트랙트 주소와 [ABI](https://docs.soliditylang.org/en/latest/abi-spec.html))가 있는 경우 ContractKit를 사용하여 배포된 스마트 컨트랙트와 상호 작용할 수 있습니다. 이를 위해 새로운 `web3` 컨트랙트 인스턴스를 초기화합니다. 그런 다음 컨트랙트 인스턴스의 함수를 호출하여 상태를 읽거나 트랜잭션을 전송하여 컨트랙트를 업데이트할 수 있습니다. 아래 몇 가지 코드의 일부분을 참고하세요. 보다 포괄적인 예시는 컨트랙트 코드 배포 예제의 [사용자 정의 컨트랙트와 상호 작용하기](../walkthroughs/hello-contract-remote-node.md#interacting-with-custom-contracts) 섹션을 참고하세요.

```ts
let contract = new kit.web3.eth.Contract(ABI, address)       // Init a web3.js contract instance
let name = await instance.methods.getName().call()       // Read contract state call

const txObject = await instance.methods.setName(newName) // Encoding a transaction object call to the contract
await kit.sendTransactionObject(txObject, { from: account.address }) // Send the transaction
```

## 수수료가 유리한 경우에만 CELO 판매하기

```ts
// This is at lower price I will accept in cUSD for every CELO
const favorableAmount = 100
const amountToExchange = kit.web3.utils.toWei('10', 'ether')
const oneGold = kit.web3.utils.toWei('1', 'ether')
const exchange = await kit.contracts.getExchange()

const amountOfcUsd = await exchange.quoteGoldSell(oneGold)

if (amountOfcUsd > favorableAmount) {
  const goldToken = await kit.contracts.getGoldToken()
  const approveTx = await goldToken.approve(exchange.address, amountToExchange).send()
  const approveReceipt = await approveTx.waitReceipt()

  const usdAmount = await exchange.quoteGoldSell(amountToExchange)
  const sellTx = await exchange.sellGold(amountToExchange, usdAmount).send()
  const sellReceipt = await sellTx.waitReceipt()
}
```

## 내 계좌에 있는 cUSD로 살 수 있는 모든 CELO를 사기

```ts
const stableToken = await this.contracts.getStableToken()
const exchange = await this.contracts.getExchange()

const cUsdBalance = await stableToken.balanceOf(myAddress)

const approveTx = await stableToken.approve(exchange.address, cUsdBalance).send()
const approveReceipt = await approveTx.waitReceipt()

const goldAmount = await exchange.quoteUsdSell(cUsdBalance)
const sellTx = await exchange.sellDollar(cUsdBalance, goldAmount).send()
const sellReceipt = await sellTx.waitReceipt()
```
