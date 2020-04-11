# FlexID DID Method Specification v0.1

FlexID is an implementation of the Sidetree protocol built on top of the Algorand blockchain. For information about Sidetree, look at the [Sidetree specification](https://github.com/decentralized-identity/sidetree/blob/master/docs/protocol.md).

### Method Syntax

A FlexID DID _MUST_ begin with `did:flex` (case-sensitive). It has the format of `did:flex:<network_name>:<unique_id>` where:

- `network_name` is an optional parameter which can be one of `testnet` or `mainnet`
- `unique_id` is a `sha256` hash of the [Create Payload](#) operation specified later.

The following are all examples of valid FlexID DID:

- `did:flex:mainnet:EiAsiomzmhP3j6La_8FX6O-qdk8pUbD7K87n46tT909wHg`
- `did:flex:testnet:EiB2A-zsP2Vy0-8yqqdLEmgvnendvuNGgcOjXCJ303ttsQ`
- `did:flex:EiBRRnOpwxjgtpi1sMZ3wVhyAAPqUNtFoE0pFP45sckBEA`

If no network is specified in the prefix, it is assumed to be a testnet DID.

### Method Operations

FlexID builds on and uses the Sidetree core specification. There are five operations used to Create, Resolve, Update, Delete and Recover DIDs. These can be called through HTTP requests to a Flex Sidetree node. Every operation _MUST_ follow the following syntax

```javascript
{
    protected: "base64 encoded header",
    payload: "base64 encoded payload of the operation",
    signature: "base64 encoded signature(encoded header, encoded payload)",
}
```

##### Create

The Create operation is used to register a new DID.

The header of a create operation _MUST_ follow the following syntax:

```javascript
{
    operation: "create",
    kid: "key id to use for signing",
    alg: "algorithm used for signing eg. Ed25519"
}
```

The payload of a create operation _MUST_ be a DID Document Model without the `id` property and without the `controller` property for the public keys associated with that DID. For more information about DID Documents, look at the [DID Core Spec](https://www.w3.org/TR/did-core/).

The create request is called by submitting the operation to `POST /requests` on the Sidetree node.

##### Resolve

The Resolve operation is used to fetch the DID Document associated with a DID.

Resolution _MUST_ happen through a Sidetree node to ensure all patches to the DID Documents are applied properly.

A resolution request is called by submitting a `GET /:did` request to the Sidetree node where `did` is the DID you want to resolve.

##### Update

The Update operation is used to provide patches to add or remove keys, key capabilities, and service endpoints to a DID.

The header of an update operation _MUST_ follow the following syntax:

```javascript
{
    operation: "update",
    kid: "key id to use for signing",
    alg: "algorithm used for signing eg. Ed25519",
}
```

The payload of an update operation _MUST_ follow the following syntax:

```javascript
{
    didUniqueSuffix: "unique DID id",
    previousOperationHash: "hash of last operation on this DID",
    patches: []
}
```

where `patches` is an array of supported patches to apply. Currently we support the following patches:

- `add-public-keys`
- `remove-public-keys`
- `add-authentication`
- `remove-authentication`
- `add-assertion-method`
- `remove-assertion-method`
- `add-capability-delegation`
- `remove-capability-delegation`
- `add-capability-invocation`
- `remove-capability-invocation`
- `add-key-agreement`
- `remove-key-agreement`
- `add-service-endpoint`
- `remove-service-endpoint`

The `patches` array includes objects of the following syntax:

```javascript
{
    action: "supported patch action",
    publicKeys: []
}
```

where `publicKeys` is either an array of keys or key ids based on whether it's an `add-*` or `remove-*` patch.

Full example of update payload:

```javascript
{
  didUniqueSuffix: "EiAsiomzmhP3j6La_8FX6O-qdk8pUbD7K87n46tT909wHg",
  previousOperationHash: "EiAsiomzmhP3j6Lasdf2838FJsnf39DDa820DMP",
  patches: [
    {
      action: "add-public-keys",
      publicKeys: [
        {
          id: "#newKey",
          usage: "signing",
          type: "Ed25519VerificationKey2018",
          publicKeyBase58: "4wHaE65mVBaLQvkRfubmTaetW6wkp5KBB5oNpAwVp8CV"
        },
        {
          id: "#newKey3",
          usage: "signing",
          type: "Ed25519VerificationKey2018",
          publicKeyBase58: "6x31cdTfCwem9ZwmfYSxKNYDp2v9Z8vj5X7NksuMXzvY"
        }
      ]
    },
    {
      action: "remove-public-keys",
      publicKeys: [
        "#primary"
      ]
    }
  ]
}
```

The update request is called by submitting the operation to `POST /requests` on the Sidetree node.

##### Delete

The Delete operation essentially deletes a DID by marking it no longer in use. Sidetree will no longer resolve it, but the information that has been put on chain in the past will continue to stay there, although useless.

The header of a delete operation _MUST_ follow the following syntax:

```javascript
{
    operation: "delete",
    kid: "key id to use for signing",
    alg: "algorithm used for signing e.g. Ed25519"
}
```

The payload of a delete operation _MUST_ follow the following syntax:

```javascript
{
  didUniqueSuffix: "unique DID id";
}
```

The delete request is called by submitting the operation to `POST /requests` on the Sidetree node.

##### Recover

The Recover operation is used to regain control of a DID you have lost the keys to. You _MUST_ have access to a recovery key specified in your DID Document to be able to Recover your account.

The header of a recover operation _MUST_ follow the following syntax:

```javascript
{
    operation: "recover",
    kid: "key id for recovery key",
    alg: "algorithm used for signing e.g. Ed25519",
}
```

The payload of a recover operation _MUST_ follow the following syntax:

```javascript
{
    didUniqueSuffix: "unique DID id",
    newDidDocument: "new DID Document model"
}
```

The recover request is called by submitting the operation to `POST /requests` on the Sidetree node.
