# did:polygonid Method Specification

Polygon ID - is an implementation of the [iden3 protocol](https://docs.iden3.io/protocol/spec/). The protocol is compatible with any Ethereum Virtual Machine (EVM) blockchain. Polygon ID aims to maximize the privacy of the identity holder by leveraging Zero Knowledge Proofs (ZKP) technology. The protocol enables users to generate ZK proof responses on Verifier request using the [ZK query language](https://0xpolygonid.github.io/tutorials/verifier/verification-library/zk-query-language/).

For an overview of Polygon ID please read our tutorials page: [https://0xpolygonid.github.io/tutorials/](https://0xpolygonid.github.io/tutorials/)

## Method Specific Identifier

The namestring that shall identify this DID method is: `polygonid`

A DID that uses this method MUST begin with the following prefix: *`did:polygonid`*

Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is as follows:

---

The method identifier is composed of a network identifier (“polygon”) and a specific identifier inside the network, such as "main", “mumbai” in Polygon network.

The identity identifier is base58 encoded [iden3 specific identifier](https://docs.iden3.io/getting-started/identity/identifier/) that is permanent, unique, and deterministically calculated from [the Genesis ID](https://docs.iden3.io/protocol/spec/#genesis-id).

There is a special case when identity is not linked to any network. That means that identity cannot change it's state and because of this it's marked as "readonly".

```
polygonid-did = "did:polygonid:" polygonid-specific-idstring
polygonid-specific-idstring = [polygonid-blockchain ":" polygonid-networkID ":" ] polygonid-identifier
polygonid-blockchain = "polygon" / "eth" / "readonly"
polygonid-networkID = "main" / "mumbai" / "goerli"
polygonid-identifier = 42*43 BASE58
```

### Example

A valid polygonid DID:

```
did:polygonid:polygon:main:2pzr1wiBm3Qhtq137NNPPDFvdk5xwRsjDFnMxpnYHm
did:polygonid:polygon:mumbai:2qCU58EJgrELNZCDkSU23dQHZsBgAFWLNpNezo1g6b
did:polygonid:readonly:2mbH5rt9zKT1mTivFAie88onmfQtBU9RQhjNPLwFZh // readonly
```

## DID Document


Example:

```json
{
        "@context": [
            "https://www.w3.org/ns/did/v1",
            "https://schema.iden3.io/core/jsonld/auth.jsonld"
        ],
        "id": "did:polygonid:polygon:mumbai:2qL4AhwQThPWJ8QH9mxH41CDgKj9chrHN92azh5wkx",
        "authentication": [
            {
                "id": "did:polygonid:polygon:mumbai:2qL4AhwQThPWJ8QH9mxH41CDgKj9chrHN92azh5wkx?state=f3ed43e7fa950d746331b143bf16eae2dd97a1876704cbb8c620913421ea2f12",
                "type": "Iden3StateInfo2023",
                "blockchainAccountId": "80001:0x134B1BE34911E39A8397ec6289782989729807a4",
                "published": true,
                "info": {
                    "id": "did:polygonid:polygon:mumbai:2qL4AhwQThPWJ8QH9mxH41CDgKj9chrHN92azh5wkx",
                    "state": "f3ed43e7fa950d746331b143bf16eae2dd97a1876704cbb8c620913421ea2f12",
                    "replacedByState": "0000000000000000000000000000000000000000000000000000000000000000",
                    "createdAtTimestamp": "1677691329",
                    "replacedAtTimestamp": "0",
                    "createdAtBlock": "32582097",
                    "replacedAtBlock": "0"
                },
                "global": {
                    "root": "d5a1afd7dd885ca97493c29640c8d43bc1af1ce165186974a42ddddfdea4182d",
                    "replacedByRoot": "0000000000000000000000000000000000000000000000000000000000000000",
                    "createdAtTimestamp": "1677691329",
                    "replacedAtTimestamp": "0",
                    "createdAtBlock": "32582097",
                    "replacedAtBlock": "0"
                }
            }
        ]
    }
```

## JSON-LD Context

---

The polygonid did method uses additional JSON-LD types.

The JSON-LD vocabulary is stored in:

```
https://schema.iden3.io/core/jsonld/auth.jsonld
```

Context contains `AuthBJJCredential`(Operational key)  and `Iden3StateInfo2023` types.

```
https://schema.iden3.io/core/jsonld/iden3proofs.jsonld
```

Context contains `Iden3SparseMerkleTreeProof` and `BJJSignature2021` proofs types used with polygonID identity.

## Basic operations

Each identity has a unique DID that is determined by the initial identity state (hash of its Merkle Tree Roots). This identifier is called `[Genesis ID](https://docs.iden3.io/protocol/spec/#genesis-id)`, under which the initial claims are placed (those that are contained in the initial state of the identity).

### Create

The creation of the identity is done by creating a [Genesis ID](https://docs.iden3.io/protocol/spec/#genesis-id). While an identity does not add, update or revoke claims after the Genesis State, its identity state does not need to be published on the blockchain, and the `Genesis Claims` can be verified directly against the `Genesis ID`. This is because the Genesis ID is built by the Merkle Root that holds those claims.

### State transition (Update)

Adding new claims (claim statements or key claims), updating them through versions and revoking need to be done according to the identity [state transition function](https://docs.iden3.io/protocol/spec/#identity-state-transition-function).
To ensure this, we use zero-knowledge proofs in a way that when an identity is publishing a new state to the smart contract, it also sends a zero-knowledge proof, proving that it is satisfied following the protocol rules.

**Deactivate**

When an identity revokes all the `claims` of the type `operational key authorization`, it is considered [deactivated](https://docs.iden3.io/protocol/spec/#identity-revocation) as this identity can no longer create proofs.

### Type of identity

- *Regular* - identity generated by the user or organization off-chain
- *On-chain identity* - identity created and managed by a smart contract. This type of identities can behave as on-chain issuer. Allowing any dAPP to issue credentials to it’s users in a trustless manner.

### Resolve

Abstract algorithm for resolving [(read operation).](https://www.w3.org/TR/did-core/#resolution)

Contract address for resolving the did (mumbai network): `0x134B1BE34911E39A8397ec6289782989729807a4`

`PolygonID did driver` accepts DID in URI format. Currently, we support three URI formats:

1. `did:polygonid:polygon:mumbai:2qHaobUk3dBVCkox8HtJezfWLDnGrBPufieEJ5A5o8`
2. `did:polygonid:polygon:mumbai:2qHaobUk3dBVCkox8HtJezfWLDnGrBPufieEJ5A5o8?state=<hex_of_state>`
3. `did:polygonid:polygon:mumbai:2qHaobUk3dBVCkox8HtJezfWLDnGrBPufieEJ5A5o8?gist=<hex_of_gist_state>`
Each of these formats has a different `read` algorithm. But they all have a certain common logic of presentation.
4. Common steps for all formats of DID.<br/>
1.1 Get a method from DID (`polygonid|iden3`).<br/>
1.2 Get a blockchain name from DID (`polygon|eth`).<br/>
1.2 Get chain ID from DID. For `polygon` chain we support `main|mumbai` networks for `eth` chain `main|goerly` networks).<br/>
1.3 Verify data that was encoded on the identification (`2qHaobUk3dBVCkox8HtJezfWLDnGrBPufieEJ5A5o8`) matches on parsed DID info.<br/>
1.4 Find resolver by pair of chain name and blockchain id.<br/>
5. Step 5:<br/>
5.1 `Read` operation for the simple format of DID (`did:polygonid:polygon:mumbai:2qHaobUk3dBVCkox8HtJezfWLDnGrBPufieEJ5A5o8`). This format means that the resolver should read the latest information about ID and GIST from the contract. If ID is the genesis one or doesn't exist on the contract, resolver will return only GIST info.<br/>
    5.1.1 Call `getGISTRoot` from the contract to get the latest GIST state.<br/>
    5.1.2 Call `getGISTRootInfo` with GIST from the step above to get the latest information about GIST.<br/>
    5.1.3 Call `getStateInfoById` with the user ID to get the latest information about the user’s state. If it does not exist return an empty object<br/>
    ```json
    "info": {
        "id": "did:polygonid:polygon:mumbai:2qHaobUk3dBVCkox8HtJezfWLDnGrBPufieEJ5A5o8",
        "state": "0a4f612419d84ca6ddaa377ec3f2411aba4bc6a3caac6996bc726790a0693424",
        "replacedByState": "0000000000000000000000000000000000000000000000000000000000000000",
        "createdAtTimestamp": "1677692829",
        "replacedAtTimestamp": "0",
        "createdAtBlock": "32582803",
        "replacedAtBlock": "0"
    },
    "global": {
        "root": "6b16687215c6cdacc3134968d222ce12d1b3a47f84075c4e1cb6074782f14809",
        "replacedByRoot": "0000000000000000000000000000000000000000000000000000000000000000",
        "createdAtTimestamp": "1677846792",
        "replacedAtTimestamp": "0",
        "createdAtBlock": "32655210",
        "replacedAtBlock": "0"
    }
    ```
5.2 `did...#state=<state>` Resolve DID document by committed `state`. In some cases, we should get historical DID to validate the information that was replaced at the current time. Since we can’t get GIST by the user’s ID, global info will not exist in the resolution document.<br/>
    5.2.1 Call `getStateInfoByState` from the contract to get information about the user’s state by state.<br/>
    5.2.2 Verify that this state is the user’s state.<br/>
```json
    "info": {
      "id": "did:polygonid:polygon:mumbai:2qHaobUk3dBVCkox8HtJezfWLDnGrBPufieEJ5A5o8",
      "state": "0a4f612419d84ca6ddaa377ec3f2411aba4bc6a3caac6996bc726790a0693424",
      "replacedByState": "0000000000000000000000000000000000000000000000000000000000000000",
      "createdAtTimestamp": "1677692829",
      "replacedAtTimestamp": "0",
      "createdAtBlock": "32582803",
      "replacedAtBlock": "0"
    }
```
 5.3 `did...#gist=<state>` Resolve state by gist state. Identities can exist under a global state for increased security and anonymity. In some cases, we should understand that it may exist or not exist for some user cases under the global target state.<br/>
    5.3.1 Call `getGISTProofByRoot` with GIST root (key) and users ID (value), for get exist or not exist proof.<br/>
    5.3.2 Call `getGISTRootInfo` with `root` from the step above to get GIST information.<br/>
    5.3.3 If the proof has `exists = true`, execute the next state. If `exists = false` return information only about GIST state.<br/>
    5.3.4 Call `getStateInfoByState` with the value from proof to return information about the user’s state.<br/>
```json
"info": {
    "id": "did:polygonid:polygon:mumbai:2qHaobUk3dBVCkox8HtJezfWLDnGrBPufieEJ5A5o8",
    "state": "0a4f612419d84ca6ddaa377ec3f2411aba4bc6a3caac6996bc726790a0693424",
    "replacedByState": "0000000000000000000000000000000000000000000000000000000000000000",
    "createdAtTimestamp": "1677692829",
    "replacedAtTimestamp": "0",
    "createdAtBlock": "32582803",
    "replacedAtBlock": "0"
},
"global": {
    "root": "6b16687215c6cdacc3134968d222ce12d1b3a47f84075c4e1cb6074782f14809",
    "replacedByRoot": "0000000000000000000000000000000000000000000000000000000000000000",
    "createdAtTimestamp": "1677846792",
    "replacedAtTimestamp": "0",
    "createdAtBlock": "32655210",
    "replacedAtBlock": "0"
}
```

JSON result description: [https://github.com/iden3/claim-schema-vocab/blob/main/core/vocab/state-info.md](https://github.com/iden3/claim-schema-vocab/blob/main/core/vocab/state-info.md)

6. Build representation document:
After fetching the gist and the user’s state it is possible to build a representation document.<br/>
  6.1 Create an object with the following fields:
    ```
    struct {
       id
       type
       blockchainAccountId
       published
       latest
       global
    }
    ```
  6.2 In the `latest` field, put information about the user’s state. In the `global` field, put information about the GIST state.<br/>
  6.3 Fill `id` filed in the following format `<did>#state=<requested_state|latest_state>`.
        `requested_state` - use the `state` from the request if the request had `#sate=...`
        `latest_state` - use the state from the resolver response.<br/>
  6.4 Add `Iden3StateInfo` type to `type` field.<br/>
  6.5 Fill `blockchainAccountId` in the following format  `<chain_id>:<resolver_contract>`.<br/>
  6.6 Set `true` in `published` if the user’s state was been published to the smart contract.<br/>
  6.7 Create an array `authentication` and put the struct from step 6.1 in this array.<br/>
  6.8 Put the array `authentication` to `didDocument`. Also, add `@context: ["https://www.w3.org/ns/did/v1", "https://schema.iden3.io/core/jsonld/auth.jsonld"]` to this `didDocument`.<br/>
  6.9 Build representation:<br/>

```json
{
	"@context": ["https://w3id.org/did-resolution/v1"],
	"didDocument": {},
	"didResolutionMetadata": {},
	"didDocumentMetadata": {}
}
```

## Security and Privacy Considerations

There are several security and privacy considerations that implementers recommended to take into consideration when implementing this specification:

**Data Forgery Prevention**

Our DID method prevents forgery and falsification through the usage of Digital Signatures, Merkle Tree Proofs, and Zero Knowledge Proofs (ZKPs) such that only the identity owner can issue credentials or present credentials to third parties. The identity owner can choose to A) Only issue credentials with digital signatures or; B) Issue claims with all both digital signatures and Merkle Tree Proofs. With digital signatures, only the controller of the DID Document is capable of issuing the credentials. When adding merkle tree proofs this adds an extra layer of forgery prevention by allowing the holder of the claim to demonstrate that 1) [That the credential is currently valid](https://docs.iden3.io/protocol/spec/#prove-that-the-claim-is-currently-valid), and 2) [The credential has not been revoked recently](https://docs.iden3.io/protocol/spec/#prove-that-the-claim-hasnt-been-revoked-recently). Both of [these elements are used to create a zero knowledge proof](https://docs.iden3.io/protocol/spec/#zero-knowledge-proof-of-valid-credentials) which is presented the verifier of the claim (unless selective disclosure of credential data is used).

**Eavesdropping attacks**

Eavesdropping attacks need to be mitigated by the usage of a secure communication channel (secured with TLS or similar means), since we use a message based communication protocol that is not natively encrypted.

**Cryptographic Agility**

The Polygon ID DID method supports EDDSA on the [Baby JubJub curve](https://github.com/iden3/iden3-docs/blob/master/source/docs/Baby-Jubjub.pdf) and [Sparse Merkle Tree](https://docs.iden3.io/protocol/spec/#merkle-tree) Proofs for the “signing” of credentials. For the generation of the zero knowledge proofs we use zkSNARKs with the Groth16 proving scheme. We may add other “signing” and “proving” algorithms in the future.

**Keep DID Keys safe**

The DID method includes support for key rotation, however if the key is compromised then an attacker may rotate keys, and block the original owner from making any changes. Care should be used when handling private keys.

**Keep personal data safe**

The syntax and construction of the Polygon ID DID and its associated DID Document helps to ensure that no Personally Identifiable Information (PII) or other personal data is exposed by these constructs: 

- Only the state of identity is required to be published on-chain. In case of on-chain identity all the data will be available in the smart-contract.
- Identifiers in Polygon ID do not reveal specific personal information about the identity holder, and the holder can choose to have unique random identifiers for each new interaction with verifiers.
- We also implemented zero-knowledge proofs such that interactions between the user and verifier reveal the minimal amount of data possible.

**Additional Privacy and Security Recommendations:**

Implementers are strongly encouraged to review the following: 

- The Privacy Considerations section of the DID Implementation Guide: [https://w3c.github.io/did-imp-guide/#privacy-considerations](https://w3c.github.io/did-imp-guide/#privacy-considerations).
- The Privacy Considerations section of the Decentralized Identifiers (DIDs) (DID-CORE) specification: [https://www.w3.org/TR/did-core/#privacy-considerations](https://www.w3.org/TR/did-core/#privacy-considerations).
- The Privacy Considerations section of the DID Implementation Guide: [https://w3c.github.io/did-imp-guide/#privacy-considerations](https://w3c.github.io/did-imp-guide/#privacy-considerations).
- The Privacy Considerations section of the Decentralized Identifiers (DIDs) (DID-CORE) specification: [https://www.w3.org/TR/did-core/#privacy-considerations](https://www.w3.org/TR/did-core/#privacy-considerations).

---

## References

---

- [https://w3c-ccg.github.io/did-spec/](https://w3c-ccg.github.io/did-spec/)
- [https://idocs.iden3.io/#/core/spec/spec?id=identifier-format](https://idocs.iden3.io/#/core/spec/spec?id=identifier-format)
- [https://idocs.iden3.io/#/core/spec/spec](https://idocs.iden3.io/#/core/spec/spec?id=identifier-format)
- [https://0xpolygonid.github.io/tutorials/](https://0xpolygonid.github.io/tutorials/)
