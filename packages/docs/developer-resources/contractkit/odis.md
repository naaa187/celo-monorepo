# ODIS를 사용하여 온체인 식별자 쿼리하기 (ODIS)

이 가이드에서는 ContractKit를 사용하여 전화 번호가 주어진 온체인 식별자를 쿼리하는 방법을 안내합니다. ODIS에 대한 자세한 내용은 [이 개요 문서](../../celo-codebase/protocol/identity/phone-number-privacy.md)를 참조하십시오. Celo의 주요 기능 중 하나는 전화 번호를 Celo 주소에 연결하는 기능입니다. 이 기능은 Celo 사용자에게 편리한 결제 경험을 제공합니다. 전화 번호를 주소에 매핑하려면 먼저 주어진 전화 번호에 대한 온체인 식별자를 검색해야 합니다. 이 식별자를 사용하여 주소를 온체인에서 조회할 수 있습니다.

{% hint style="info" %}
ODIS 요청은 트랜잭션 기록 및 잔액에 따라 요금이 제한됩니다. 조회를 수행하는 계정에 잔액이 있고 네트워크에서 트랜잭션을 수행했는지 확인하십시오. 할당량 초과 오류가 발생하면, 이는 쿼리하고있는 계정에서 더 많은 트랜잭션을 전송해야 함을 나타냅니다.
{% endhint %}

ODIS에는 두 가지 방법이 있습니다:

1. `getPhoneNumberIdentifier` - 전화 번호에 대한 식별자를 쿼리하고 계산합니다.
2. `getContactMatches` - 사용자 간의 상호 연결 찾기

## 인증

두 가지 방법 모두 ODIS 서버에 대한 인증이 필요하며, 주 지갑 키나 지갑 키와 연결된 데이터 암호화 키(DEK)를 통해 인증을 수행할 수 있습니다. 이 작업은 지갑 키의 WalletKeySigner 또는 DEK의 EncryptionKeySigner일 수 있는 AuthSigner에 의해 관리됩니다. DEK 방법은 사용자가 자금을 관리하는 동일한 키에 액세스할 필요가 없기 때문에 선호됩니다. [여기서 DEK에 대해 자세히 알아볼 수 있습니다.](./data-encryption-key.md)

원시 개인 키를 전달하여 AuthSigner에 EncryptionKeySigner를 사용할 수 있습니다.

```ts
const authSigner: AuthSigner = {
  authenticationMethod: OdisUtils.Query.AuthenticationMethod.ENCRYPTION_KEY,
  rawKey: privateDataKey,
}
```

또는 계정의 잠금이 해제된 contractkit 인스턴스를 전달하여 WalletKeySigner를 사용할 수도 있습니다.

```ts
const authSigner: AuthSigner = {
  authenticationMethod: OdisUtils.Query.AuthenticationMethod.WALLET_KEY,
  contractKit,
}
```

## 서비스 컨텍스트

ServiceContext 객체는 ODIS 엔드포인트 URL과 ODIS 공개키(위와 동일)를 제공합니다.

```ts
const serviceContext: ServiceContext = {
  odisUrl,
  odisPubKey,
}
```

각 환경의 ODIS 엔트포인트 URL은 다음과 같습니다:
| Environment | Key |
|---|---|
| Alfajores Staging | https://us-central1-celo-phone-number-privacy-stg.cloudfunctions.net |
| Alfajores | https://us-central1-celo-phone-number-privacy.cloudfunctions.net |
| Mainnet | https://us-central1-celo-pgpnp-mainnet.cloudfunctions.net |

각 환경의 ODIS 공개키는 다음과 같습니다:
| Environment | Key |
|---|---|
| Alfajores Staging | 7FsWGsFnmVvRfMDpzz95Np76wf/1sPaK0Og9yiB+P8QbjiC8FV67NBans9hzZEkBaQMhiapzgMR6CkZIZPvgwQboAxl65JWRZecGe5V3XO4sdKeNemdAZ2TzQuWkuZoA |
| Alfajores | kPoRxWdEdZ/Nd3uQnp3FJFs54zuiS+ksqvOm9x8vY6KHPG8jrfqysvIRU0wtqYsBKA7SoAsICMBv8C/Fb2ZpDOqhSqvr/sZbZoHmQfvbqrzbtDIPvUIrHgRS0ydJCMsA |
| Mainnet | FvreHfLmhBjwxHxsxeyrcOLtSonC9j7K3WrS4QapYsQH6LdaDTaNGmnlQMfFY04Bp/K4wAvqQwO9/bqPVCKf8Ze8OZo8Frmog4JY4xAiwrsqOXxug11+htjEe1pj4uMA |

## 전화번호 식별자 쿼리하기

이 호출은 할당량을 사용합니다. 사용자가 할당량을 초과하여 실행하는 경우, 트랜잭션을 전송하여 더 많은 할당량을 "구매"하도록 사용자에게 안내하는 것이 좋습니다. 이 방법은 서비스에서 검색한 페퍼와 더불어 해당 페퍼와 전화 번호를 사용하여 생성된 온체인 식별자를 반환합니다.

### BLS 블라인딩 클라이언트

It's important for user privacy that the ODIS servers don't have the ability to view the raw phone number. Before making the request, the library first blinds the phone number using a BLS library. This prevents the ODIS from being able to see the phone number but still makes the resulting signature recoverable to the original phone number. The blinding client is written in [Rust](https://github.com/celo-org/celo-threshold-bls-rs) and compiled to Web Assembly, which is not compatible with React native. If you choose not to pass in a `BLSBlindingClient` it will default to the Web Assembly version. You may create a `ReactBlindingClient` by calling the constructor with the ODIS public key:

```ts
const blsBlindingClient = new ReactBlsBlindingClient(odisPubKey)
```

Or use the `WasmBlsBlindingClient` if your runtime environment supports Web Assembly:

```ts
const blsBlindingClient = new WasmBlsBlindingClient(odisPubKey)
```

Now you're ready to get the phone number identifier. `OdisUtils.PhoneNumberIdentifier.getPhoneNumberIdentifier` [documentation can be found here](../identity/reference/modules/_odis_phone_number_identifier_.md#getphonenumberidentifier).

The response will be [an object](../identity/reference/interfaces/_odis_phone_number_identifier_.phonenumberhashdetails.md) with the original phone number, the on-chain identifier (phoneHash), and the phone number's pepper.

You can view an example of this call in [our mobile project here](https://github.com/celo-org/wallet/blob/master/packages/mobile/src/identity/privateHashing.ts).

## Matchmaking

Instead of querying for all the user's contact's peppers and consuming the user's quota, it's recommended to only query the pepper before it's actually used (ex. just before sending funds). However, sometimes it's helpful to let your users know that they have contacts already using the Celo network. To do this, you can make use of the matchmaking interface. Given two phone numbers, it will let you know whether the other party has also registered on the Celo network with this identifier. `OdisUtils.Matchmaking.getContactMatches` [documentation can be found here](reference/modules/_identity_claims_account_.md).

The response will be a subset of the input `e164NumberContacts` that are matched by the matchmaking service.

You can view an example of this call in [our mobile project here](https://github.com/celo-org/wallet/blob/master/packages/mobile/src/identity/matchmaking.ts).
