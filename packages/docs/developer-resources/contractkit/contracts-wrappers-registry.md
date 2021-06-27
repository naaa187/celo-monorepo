# Celo Core Contracts. Wrappers / Registry

## CELO & cUSD와 상호작용하기

Celo 블록체인에는 CELO와 cUSD(stableToken)라는 두 개의 초기 코인이 있습니다.
둘 다 ERC20 표준을 구현하며 이들과 상호 작용하는 것은 다음과 같이 간단합니다:

```ts
const goldtoken = await kit.contract.getGoldToken()

const balance = await goldtoken.balanceOf(someAddress)
```

자금을 보내려면:

```ts
const oneGold = kit.web3.utils.toWei('1', 'ether')
const tx = await goldtoken.transfer(someAddress, oneGold).send({
  from: myAddress,
})

const hash = await tx.getHash()
const receipt = await tx.waitReceipt()
```

cUSD와 상호 작용하려면 방법은 동일하지만 Contract가 다릅니다:

```ts
const stabletoken = await kit.contract.getStableToken()
```

## 다른 Celo Contracts와 상호작용하기

GoldToken과 StableToken을 제외하고, 많은 core contract가 있습니다.

현재 다음을 위한 컨트랙트 래퍼가 있습니다:

- Accounts
- Attestations
- BlockchainParameters
- DobleSigningSlasher
- DowntimeSlasher
- Election
- Escrow
- Exchange (Uniswap kind exchange between Gold and Stable tokens)
- GasPriceMinimum
- GoldToken
- Gobernance
- LockedGold
- Reserve
- SortedOracles
- Validators
- StableToken

## Contract 주소에 대한 참고사항

Celo Core Contracts 주소는 `레지스트리` 컨트랙트을 통해 확인할 수 있습니다.
그것이 바로 `키트`가 그것들을 얻는 방법입니다.

레지스트리 api는 다음과 같이 액세스할 수 있습니다:

```ts
const goldTokenAddress = await kit.registry.addressFor(CeloContract.GoldToken)
```

## web3 contract wrapper에 접근하기

일부 사용자는 web3 네이티브 계약 래퍼에 액세스하기를 원할 수 있습니다. 실수를 피하기 위해 Celo 계약을 대신 사용할 것을 권장합니다.
이를 위해 다음을 수행할 수 있습니다:

```ts
const web3Exchange = await kit._web3Contracts.getExchange()
```

NAT은 모든 웹3 계약에 네이티브 래퍼를 노출합니다.

전체 목록은 다음과 같습니다:

- Accounts
- Attestations
- BlockchainParameters
- DoubleSigningSlasher
- DowntimeSlasher
- Election
- EpochRewards
- Escrow
- Exchange
- FeeCurrencyWhiteList
- GasPriceMinimum
- GoldToken
- Governance
- LockedGold
- Random
- Registry
- Reserve
- SortedOracles
- StableToken
- Validators

## 디버깅

키트를 디버그해야 하는 경우 잘 알려진 디버그 노드 라이브러리를 사용합니다.

따라서 환경 변수를 DEBUG로 설정합니다:

```bash
DEBUG="kit:*,
```
