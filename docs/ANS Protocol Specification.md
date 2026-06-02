<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

# `did:ans` Method Specification

Version 1.0.0

## Status of This Document

This document specifies the `did:ans` DID method. It is intended to be read together with the [W3C DID Core Recommendation](https://www.w3.org/TR/did-core/).

This specification defines ANS as a DID **method** for agent-native identity, service discovery, address identification, delegation-aware control, and model-fingerprint-aware bindings in blockchain and cross-domain digital systems.

The latest version of this document is available from the [ANS repository path](https://github.com/wolfbrother/did-ans/blob/main/doc/en/ANS%20Protocol%20Specification.md).

## 1. Introduction

`did:ans` is a decentralized identifier method for **intelligent agents**, especially software agents, AI service agents, and model-backed agents that need:

- stable decentralized identity,
- verifiable controller keys,
- associated service and address discovery,
- delegated authority chains,
- and optional bindings to large-model fingerprints or model-derived identities.

`did:ans` is optimized for **agent-centric identity**. In this version of the method, the DID subject represents an intelligent agent. It commonly represents:

- a software agent,
- an AI service agent,
- a model-backed agent,
- or another intelligent agent deployment.

The method is designed to support not only basic authentication, but also:

- address identification across networks and domains,
- service endpoint discovery,
- delegation and recovery semantics,
- native semantic agent description,
- and model-fingerprint-aware identity assertions for AI-related subjects.

These semantics are first-class method requirements in `did:ans`, not merely optional application conventions.

## 2. Design Goals

The `did:ans` method is designed to satisfy the following goals:

1. Identify intelligent agents in a decentralized and globally unique way.
2. Support authentication, recovery, and delegated control relationships.
3. Support service and address identification for multi-network and multi-domain systems.
4. Support native semantic agent description directly in the DID Document for agent subjects.
5. Support optional large-model fingerprint bindings for AI systems, agents, or model-backed identities.
6. Preserve DID Core interoperability while retaining ANS-specific method semantics.

## 3. Why a New DID Method Is Necessary

`did:ans` is introduced because existing general-purpose DID methods do not, by themselves, require or standardize the following semantics that are central to ANS:

1. **Address identification semantics**
   `did:ans` is intended to identify an intelligent agent together with associated address information across chains, domains, or service systems.

2. **Delegation-chain semantics**
   `did:ans` requires support for delegated authority relationships, including delegation scope, delegation proofs, and optional revocation metadata.

3. **Model-fingerprint-aware identity bindings**
   `did:ans` supports identity assertions in which an AI agent, model-backed agent, or agent service deployment may expose a large-model fingerprint or related binding metadata.

4. **Service-discovery semantics**
   `did:ans` treats service discovery and resolver routing as important method-level concerns, especially for multi-network and cross-domain systems.

5. **Native semantic agent-description semantics**
   `did:ans` supports a structured agent-description object directly in the DID Document for agent-native discovery and interoperability.

6. **Recovery-aware control model**
   `did:ans` distinguishes active authentication authority from recovery authority, allowing controlled key replacement without changing the DID itself.

If these properties were left entirely to application-specific extensions, different implementations would not be guaranteed to interpret them consistently. `did:ans` therefore defines them at the method level.

## 4. Conformance

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174).

This specification conforms to the W3C DID Core data model and terminology. Where this method introduces additional properties, those properties are method-specific extensions and MUST NOT invalidate DID Core processing expectations.

## 5. Representations

The primary DID Document representation defined by this specification is JSON-LD.

When a `did:ans` DID Document is represented as JSON-LD:

- it MUST include `@context`,
- it MUST include `id`,
- and the `@context` value MUST include `https://www.w3.org/ns/did/v1`.

Implementations MAY support additional compatible representations as long as DID Core processing expectations remain satisfied.

## 6. Method Name

The name string that identifies this DID method is:

```text
ans
```

A DID produced by this method MUST begin with the lowercase prefix:

```text
did:ans
```

## 7. System Applicability

`did:ans` is intended for blockchain-integrated and cross-domain digital systems, especially where:

- intelligent agents need stable decentralized identity,
- multiple public keys or controllers may be associated with one agent,
- service endpoints must be discovered from the DID Document,
- addresses across chains, domains, or communication systems must be identified,
- delegated authority must be expressed and audited,
- and AI-related subjects may need optional model-fingerprint bindings.

The method is suitable for identity systems involving software agents, AI service agents, model-backed agents, agent wallets, and related agent service environments.

## 8. `did:ans` Identifier Syntax

### 8.1 General Structure

The `did:ans` identifier structure is defined as follows:

<img src="image/ans.png" alt="ANS Structure" style="zoom: 67%;" />

In this version of the `did:ans` method, the method-specific identifier consists of two parts:

1. a four-character semantic code; and
2. a primary identifier suffix.

The four-character semantic code is structured as:

- the first two characters indicate the **subject category**; and
- the last two characters indicate the **application domain**.

For this version of `did:ans`, the subject category is fixed to `AG`, meaning that the DID subject is an intelligent agent. This version of the method does not define other subject-category codes.

Accordingly, the semantic-code structure in this version is:

- `AG` = agent subject category
- `XX` = a two-character application-domain code defined by the relevant `did:ans` deployment, profile, registry, or governance policy

This design is intentional. It keeps the current method focused on agent-native identity, while preserving a clear and structured identifier space for different agent application domains.

### 8.2 ABNF

```abnf
ans-did = "did:ans:" ans-specific-identifier
ans-specific-identifier = semantic-code ":" suffix
semantic-code = subject-code app-domain-code
subject-code = "A" "G"
app-domain-code = 2(ALPHA / DIGIT)
suffix = 22*42(ALPHA / DIGIT)
```

Interpretation:

- `semantic-code` is a four-character method-level semantic segment.
- `subject-code` is fixed to `AG` in this version of the method and indicates that the DID subject is an intelligent agent.
- `app-domain-code` is a two-character application-domain code.
- `suffix` is the primary method-specific identifier suffix.

This restriction applies to the primary DID subject defined by this version of the `did:ans` method. Relationship positions appearing inside a DID Document, such as `controller`, delegated counterparties, or externally referenced service operators, MAY also use `did:ans` identifiers, but if they identify non-agent subjects, their subject-category codes SHOULD be distinct from `AG`.

If a broader ANS-family deployment later introduces non-agent DID subjects, those subjects SHOULD continue to follow the same `did:ans` structural pattern while using subject-category codes distinct from `AG`. This version of the method does not define those additional codes in detail, although later examples MAY use such codes illustratively to show how non-agent counterparties can be referenced.

Example structure:

```text
did:ans:AGFI:xxxxxxxxxxxxxxxxxxxxxx
```

In the example above:

- `AG` indicates an agent DID subject; and
- `FI` is an example application-domain code, such as a finance-domain agent.

### 8.3 Identifier Generation

The ANS identifier generation logic is defined as follows:

1. Select a cryptographic suite, such as `Sm2Sm3`, `Ed25519Sha256`, `Ed25519Sha256Legacy`, or another deployment-approved suite.
2. Generate a public/private key pair.
3. Encode the public key using Base58, Base32, or another ANS-compatible alphanumeric encoding profile. Any encoding used for the suffix MUST produce only letters and digits so that it conforms to the ABNF defined above.
4. Concatenate the prefix `did:ans:`, a four-character semantic code such as `AGFI`, and the encoded public key string to form the DID.

<img src="image/generateANS.png" alt="ANS Generation"  />

Cryptographic suite prefixes:

| Cryptographic Suite or Family | Encoding Prefix |
| --- | --- |
| SM2 / `Sm2Sm3` | `z` |
| Ed25519 / `Ed25519Sha256` / `Ed25519Sha256Legacy` | `e` |
| Secp256k1 | `s` |

Encoding prefixes:

| Encoding Algorithm | Encoding Prefix |
| --- | --- |
| Base58 | `f` |
| ANS-compatible alphanumeric encoding profile | `s` |
| Base32 | `t` |

### 8.4 Identifier Semantics

The `did:ans` identifier is method-specific and globally scoped within the `ans` namespace. The DID itself is immutable. Lifecycle changes affect the DID Document and associated metadata, not the DID string.

### 8.5 Cryptographic Suite Neutrality

This specification is intentionally cryptographic-suite-neutral.

- `did:ans` does not define `Ed25519` as a preferred suite.
- `did:ans` does not define `SM2` as a preferred suite.
- deployment profiles MAY adopt one suite or multiple suites simultaneously.
- new suites SHOULD be introducible at low cost by defining suite metadata, key encoding rules, and proof-verification rules without changing the overall DID method structure.

Accordingly, implementations SHOULD treat cryptographic suite selection as data-driven rather than hard-coded around a single algorithm family.

## 9. Method-Specific Characteristics

The distinctive characteristics of `did:ans` are:

1. **Agent-centric identity**
   In this version of the method, the DID subject represents an intelligent agent rather than a publication artifact or a generic non-agent controller.

2. **Address-aware identity bindings**
   The DID method can expose address information associated with chains, networks, domains, or communication endpoints.

3. **Delegation-chain awareness**
   The DID method can expose delegated authority relationships and related delegation proofs.

4. **Recovery-aware control model**
   The method supports explicit recovery authority distinct from day-to-day authentication.

5. **Native semantic agent-description support**
   The DID method may expose structured agent capability description, capability tags, and use-case examples directly in the DID Document.

6. **Model-fingerprint-aware bindings**
   The DID method may expose model fingerprint information when the subject is an AI agent, model-backed agent, or related intelligent agent deployment.

## 10. DID Document Requirements

### 10.1 Core DID Requirements

A conforming `did:ans` DID Document MUST satisfy DID Core requirements. In particular, a valid representation MUST include:

- `@context`
- `id`

The document MAY also include standard DID Core properties such as:

- `controller`
- `verificationMethod`
- `authentication`
- `assertionMethod`
- `keyAgreement`
- `capabilityInvocation`
- `capabilityDelegation`
- `service`
- `alsoKnownAs`

Only `@context` and `id` are universally required for the JSON-LD DID Document representation described here. Other DID Core properties remain optional unless additionally constrained by a particular `did:ans` deployment profile or application policy.

### 10.2 Method-Specific Requirements

For `did:ans`, a conforming DID Document SHOULD additionally include a method-specific object named `ansMetadata` when the DID identifies an intelligent agent.

The `ansMetadata` object is the primary place for method-specific identity, address, delegation, and routing semantics.

### 10.3 Method-Specific `ansMetadata`

The `ansMetadata` object MAY contain the following properties:

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| `subjectType` | string | Recommended | Subject type. In this version of the method, this value SHOULD be `agent`. |
| `identityType` | string | Optional | Deployment-specific identity classification. |
| `ttl` | integer | Optional | Resolver or cache hint in seconds. |
| `recovery` | array of string | Optional | Verification method IDs or DID URLs authorized for recovery. |
| `addressBindings` | array | Optional | Address or endpoint identification records. |
| `delegationChain` | array | Optional | Delegation records relevant to control or scoped capability. |
| `agentDescription` | object | Optional | Native semantic description of an agent subject. |
| `modelFingerprints` | array | Optional | Large-model fingerprint bindings for AI-related subjects. |
| `servicePolicy` | string | Optional | Service-discovery or routing policy label. |
| `networkScope` | string | Optional | Network, ecosystem, or domain scope label. |

### 10.4 Address Bindings

Each `addressBindings` entry MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Unique identifier for the binding entry. |
| `addressType` | string | Address class, such as `wallet`, `domain`, `endpoint`, `account`, or another deployment-defined type. |
| `network` | string | Network or namespace name, such as a chain, domain system, or communication network. |
| `address` | string | The bound address value. |
| `controller` | string | DID or DID URL asserting control over the address. |
| `purpose` | string | Intended role of the address, such as `payment`, `service`, `routing`, or `identity`. |

### 10.5 Delegation Chain

Each `delegationChain` entry MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Delegation record identifier. |
| `delegator` | string | DID or DID URL of the delegator. |
| `delegate` | string | DID or DID URL of the delegate. |
| `capability` | string | Delegated capability label. |
| `scope` | string | Scope or constraint of the delegation. |
| `created` | string | RFC 3339 creation timestamp. |
| `expires` | string | RFC 3339 expiry timestamp, if any. |
| `proof` | object | Optional delegation proof container. |
| `revoked` | boolean | Optional revocation indicator. |

The delegation chain is a method extension for expressing authority routing and MUST NOT be treated as a substitute for DID Core controller semantics.

### 10.6 Agent Description

When the DID subject is an intelligent agent, software agent, AI service agent, or model-backed agent, `ansMetadata` MAY include an `agentDescription` object.

The `agentDescription` object MAY contain the following properties:

| Property | Type | Description |
| --- | --- | --- |
| `capabilityDescription` | string | A narrative description of the agent's capabilities. |
| `capabilityTags` | array of string | A list of capability labels or phrases describing the agent's competencies. |
| `useCaseExamples` | array of string | A list of example application scenarios describing how the agent can be used. |

This object is intended to provide native semantic agent information directly in the DID Document for discovery, routing, and interoperability. It MUST NOT be treated as a substitute for cryptographic controller information or executable access policy.

### 10.7 Model Fingerprints

Each `modelFingerprints` entry MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Fingerprint entry identifier. |
| `modelProvider` | string | Model provider or operator label. |
| `modelName` | string | Model name. |
| `modelVersion` | string | Model or checkpoint version. |
| `fingerprint` | string | Large-model fingerprint value. |
| `fingerprintAlgorithm` | string | Fingerprint algorithm identifier. |
| `bindingPurpose` | string | Purpose of the binding, such as `agent-identity`, `attestation`, or `service-integrity`. |
| `created` | string | RFC 3339 creation timestamp. |

A model fingerprint is a method-specific identity binding for AI-related subjects. It MUST NOT be treated as a replacement for controller keys or DID Core authentication semantics.

### 10.8 Method-Specific `attributes`

`did:ans` MAY include an `attributes` array for application-facing descriptive metadata.

Each attribute object MAY include:

| Property | Type | Description |
| --- | --- | --- |
| `key` | string | Attribute key. |
| `desc` | string | Human-readable description. |
| `encrypt` | integer | `0` for plaintext, `1` for encrypted or protected. |
| `format` | string | Data type such as `text`, `image`, `video`, `mixture`, or another application-defined type. |
| `value` | string | Attribute value. |

`attributes` are method extensions for descriptive interoperability and MUST NOT be treated as a substitute for core controller, address, delegation, or fingerprint semantics.

### 10.9 Verification Methods

The DID Document SHOULD use DID Core `verificationMethod` rather than the legacy `publicKey` property.

Each `verificationMethod` entry SHOULD be self-describing when represented in ANS deployment profiles. In particular, ANS profiles SHOULD expose:

- `type`
- `cryptoSuite`
- `publicKeyFormat`
- one or more public-key encodings such as `publicKeyMultibase` or `publicKeyJwk`

Supported verification suites MAY include method-specific support for:

- `Ed25519VerificationKey2020`
- `SM2VerificationKey2020`
- `EcdsaSecp256k1VerificationKey2019`
- additional deployment-approved verification suites

When `cryptoSuite` is present, verifiers SHOULD use it as the primary suite selector. When `cryptoSuite` is absent in historical objects, verifiers MAY infer a compatibility suite from `type` for backward compatibility.

At least one verification method referenced from `authentication` or `assertionMethod` SHOULD be present unless the DID is permanently deactivated.

### 10.10 Services

The DID Document MAY include service endpoints. For ANS deployments, services commonly include:

- general agent identity services,
- communication or messaging endpoints,
- address-resolution services,
- and AI-agent or model-backed agent service endpoints.

If a service is intended to represent a resolution or routing service, it SHOULD include:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Service identifier. |
| `type` | string | Service type label. |
| `serviceEndpoint` | string or object | Resolution, routing, or service endpoint. |
| `version` | string | Optional service version. |
| `protocol` | string or integer | Optional transport or protocol indicator. |
| `serverType` | string or integer | Optional deployment/server type indicator. |
| `port` | integer | Optional port value. |

## 11. DID Document Metadata and DID Resolution Metadata

In DID Core, some information belongs in DID document metadata or DID resolution metadata rather than in the DID Document body itself.

Accordingly, for `did:ans`:

- `created` SHOULD normally be expressed as DID document metadata;
- `updated` SHOULD normally be expressed as DID document metadata;
- `deactivated` MUST be expressed in DID document metadata when relevant;
- and resolver status, processing status, or representation-level diagnostics SHOULD be expressed in DID resolution metadata.

This specification therefore does not require top-level `created`, `updated`, or a top-level `proof` object as universal in-document properties, even though deployments MAY include timestamps or proof material where appropriate.

When proof material is included in ANS deployment profiles, proofs SHOULD be self-describing and SHOULD carry:

- `type`
- `cryptoSuite`
- `hashAlgorithm`
- `verificationMethod`

Historical proof objects that omit `cryptoSuite` MAY still be accepted through compatibility inference from `type`, but new objects SHOULD be explicitly self-describing.

## 12. State and Control Model

`did:ans` introduces method-specific control-state expectations.

Recognized states MAY include:

- `active`
- `deactivated`

Control semantics:

1. **active**
   The DID is active and can be resolved with current controller and service information.

2. **deactivated**
   The DID has been deactivated according to method rules. Deactivation does not require historical erasure.

In addition, the method distinguishes among:

- active authentication authority,
- delegated authority,
- and recovery authority.

## 13. Method Operations

`did:ans` supports the standard DID lifecycle concepts of creation, update, and deactivation, but specializes them for agent-centric identities.

For this method, the standard DID Method operations are:

- Create
- Read
- Update
- Deactivate

In `did:ans`, the Read operation is realized through DID resolution. In addition to these standard operations, the method also defines method-specific operations such as `Recovery`, `Delegate Capability`, and `Revoke Delegation`.

### 13.1 Create

Creation establishes a new DID record.

At creation time:

- the DID MUST be unique,
- the DID Document MUST contain the DID subject in `id`,
- and any required signatures or controller authorizations MUST validate.

If recovery authority is defined at creation time, it SHOULD be distinguishable from normal authentication authority.

### 13.2 Read (Resolve)

For the `did:ans` method, the DID Method Read operation is realized through DID resolution.

Resolution returns a DID Document and DID document metadata.

Resolvers for `did:ans` SHOULD be capable of returning enough information for a relying party to:

- determine the active controller keys,
- determine available recovery and delegation semantics,
- discover service endpoints,
- identify associated addresses,
- inspect any native semantic agent-description metadata,
- and inspect any model-fingerprint bindings relevant to the subject.

### 13.3 Update

Update modifies a DID record while preserving DID identity.

For `did:ans`, updates MAY include changes to:

- verification methods,
- service endpoints,
- recovery references,
- address bindings,
- delegation records,
- and model-fingerprint metadata.

Implementations SHOULD clearly distinguish between normal updates, recovery-driven updates, and delegation-related updates.

### 13.4 Recovery

Recovery replaces or restores control according to method recovery policy.

Recovery is especially relevant when:

- controller keys are lost,
- controller keys are compromised,
- operational ownership changes,
- or governance policy requires controlled key replacement.

Recovery MUST be authorized by a valid recovery authority as defined by the DID Document or method rules.

### 13.5 Delegate Capability

`Delegate Capability` is a method-specific operation of `did:ans`.

It is used when a controller authorizes another DID, DID URL, or service-bound identity to act within a defined scope.

A delegation operation SHOULD define:

- the delegator,
- the delegate,
- the delegated capability,
- the scope,
- the validity period,
- and any attached delegation proof.

### 13.6 Revoke Delegation

`Revoke Delegation` is a method-specific operation of `did:ans`.

It is used when previously granted delegated authority must be withdrawn.

Implementations SHOULD preserve a verifiable record that the delegation existed and was later revoked, where legally and operationally appropriate.

### 13.7 Deactivate

Deactivation marks the DID as no longer active.

For `did:ans`, deactivation:

- MUST NOT require historical erasure,
- SHOULD preserve verifiability of past controller and delegation facts where legally and technically possible,
- and SHOULD be reflected in DID document metadata using `deactivated: true`.

## 14. DID Resolution

### 14.1 Resolution Output

Resolvers for `did:ans` MUST return outputs consistent with DID Core.

A conforming resolution result SHOULD include:

- `didDocument`
- `didDocumentMetadata`
- `didResolutionMetadata`

### 14.2 DID Document Metadata

For `did:ans`, DID document metadata MAY include:

| Property | Type | Description |
| --- | --- | --- |
| `created` | string | DID creation time. |
| `updated` | string | Last update time. |
| `deactivated` | boolean | Whether the DID is deactivated. |
| `controllerState` | string | Current control state. |
| `networkScope` | string | Current network or domain scope. |
| `resolvedAddresses` | array | Resolved address bindings, if exposed by the resolver. |

### 14.3 Resolution Semantics

If the DID identifies a subject with service, address, delegation, or model-fingerprint semantics, the resolver SHOULD return enough information for an external verifier to:

1. determine the current controller and authentication methods,
2. inspect recovery authority,
3. inspect available delegation-chain information,
4. discover service endpoints,
5. inspect associated address bindings,
6. and inspect any model-fingerprint bindings relevant to the subject.

## 15. DID Document Example

The following example illustrates an agent-centric `did:ans` DID Document.

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/ans/v1"
  ],
  "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2",
  "verificationMethod": [
    {
      "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#key-1",
      "type": "Ed25519VerificationKey2020",
      "controller": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2",
      "cryptoSuite": "Ed25519Sha256Legacy",
      "publicKeyFormat": "multibase",
      "publicKeyMultibase": "z6Mkexample"
    },
    {
      "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#recovery-1",
      "type": "SM2VerificationKey2020",
      "controller": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2",
      "cryptoSuite": "Sm2Sm3",
      "publicKeyFormat": "multibase",
      "publicKeyMultibase": "zSm2RecoveryExample"
    }
  ],
  "authentication": [
    "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#key-1"
  ],
  "assertionMethod": [
    "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#key-1"
  ],
  "service": [
    {
      "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#resolver",
      "type": "DIDSubResolver",
      "serviceEndpoint": "https://resolver.ans.example.org",
      "version": "1.0.0",
      "protocol": "https",
      "serverType": "public",
      "port": 443
    },
    {
      "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#agent-endpoint",
      "type": "AgentService",
      "serviceEndpoint": "https://agent.example.org/api"
    }
  ],
  "ansMetadata": {
    "subjectType": "agent",
    "identityType": "model-backed-agent",
    "ttl": 86400,
    "recovery": [
      "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#recovery-1"
    ],
    "addressBindings": [
      {
        "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#addr-wallet",
        "addressType": "wallet",
        "network": "example-network",
        "address": "addr-example-001",
        "controller": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2",
        "purpose": "identity"
      },
      {
        "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#addr-domain",
        "addressType": "domain",
        "network": "dns",
        "address": "alice.example.org",
        "controller": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2",
        "purpose": "service"
      }
    ],
    "delegationChain": [
      {
        "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#delegation-1",
        "delegator": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2",
        "delegate": "did:ans:ORFI:efDelegateExample1234567890",
        "capability": "service-management",
        "scope": "agent-service",
        "created": "2026-04-29T10:00:00Z",
        "expires": "2026-12-31T23:59:59Z"
      }
    ],
    "agentDescription": {
      "capabilityDescription": "A professional financial analysis agent that can parse enterprise disclosures, identify risk signals, summarize market developments, and generate structured investment insights for downstream systems and users.",
      "capabilityTags": [
        "financial analysis",
        "report parsing",
        "risk detection",
        "trend forecasting"
      ],
      "useCaseExamples": [
        "Analyze listed company annual reports and extract revenue, profit, and risk indicators.",
        "Review industry news and summarize possible market trend impacts.",
        "Assist investors in generating structured due-diligence briefs."
      ]
    },
    "modelFingerprints": [
      {
        "id": "did:ans:AGFI:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#model-1",
        "modelProvider": "ExampleAI",
        "modelName": "example-model",
        "modelVersion": "2026-04",
        "fingerprint": "sha256:abcdef0123456789",
        "fingerprintAlgorithm": "sha-256",
        "bindingPurpose": "agent-identity",
        "created": "2026-04-29T10:00:00Z"
      }
    ],
    "servicePolicy": "default-public-resolution",
    "networkScope": "mainnet"
  },
  "attributes": [
    {
      "key": "displayName",
      "format": "text",
      "value": "Alice Agent"
    }
  ]
}
```

The example above is intentionally multi-suite. It does not imply that `Ed25519` is preferred over `SM2`, or that `SM2` is preferred over `Ed25519`. ANS deployment profiles may use one suite, multiple suites, or introduce additional suites, as long as each object remains self-describing and verifiable.

## 16. DID Method Compliance Notes

This specification is intentionally aligned with DID Core and keeps the `did:ans` method focused on method-level semantics.

In particular:

- DID Core `verificationMethod` is used instead of legacy key-description conventions,
- transport-specific HTTP request and response payloads are not standardized because DID Core does not require a DID method to define a single transport API,
- repeated object schema descriptions are minimized where DID Core already defines the base semantics,
- and fields are classified as required, optional, metadata-level, or deployment-specific according to DID Core processing expectations.

This structure is intentional and does not remove the minimum DID Core properties required for a conforming DID Document.

## 17. Security Considerations

### 17.1 Controller Compromise

If a controller key is compromised, an attacker may attempt unauthorized updates, service redirection, or delegation abuse. Implementations SHOULD support recovery and SHOULD separate authentication from longer-term recovery authority where appropriate.

### 17.2 Delegation Abuse

Delegation relationships SHOULD be scoped, time-bounded where appropriate, and revocable. Implementations SHOULD ensure that delegated capabilities do not silently exceed the intended scope.

### 17.3 Resolver Trust

Resolvers SHOULD NOT be treated as the sole source of truth. Relying parties SHOULD verify controller keys, delegation evidence, and important metadata where possible.

### 17.4 Address and Service Misbinding

Because `did:ans` can expose service and address bindings, implementations SHOULD protect against stale, malicious, or unauthorized endpoint reassignment.

### 17.5 Model-Fingerprint Misrepresentation

Where model fingerprints are used, implementations SHOULD clarify the provenance, scope, and meaning of the fingerprint and SHOULD NOT imply stronger assurance than the fingerprinting method actually provides.

## 18. Privacy Considerations

### 18.1 Data Minimization

Identity-related metadata SHOULD be minimized to what is needed for interoperability and verification. Sensitive personal information SHOULD NOT be embedded into a DID Document unless strictly necessary.

### 18.2 Public Metadata Awareness

Because DID Documents are often publicly resolvable, method-specific properties such as `attributes`, `service`, `addressBindings`, `delegationChain`, `agentDescription`, and `modelFingerprints` SHOULD be populated with care.

### 18.3 Encrypted or Protected Attributes

Where application-specific metadata is sensitive, implementations MAY use protected references or encrypted payload strategies and indicate their use through the `attributes.encrypt` field or service-mediated access control patterns.

## 19. IANA and Registry Considerations

If `did:ans` is submitted to the W3C DID Method Registry, the registration SHOULD emphasize that the method is intended for agent-native identity, service and address identification, delegation-aware control, recovery-aware operations, and optional model-fingerprint-aware identity bindings.

## 20. Conclusion

`did:ans` is a DID method for agent-native identity, not merely a legacy DID document template or an HTTP CRUD interface. Its distinctiveness lies in method-level support for:

- agent identity,
- service and address identification,
- delegation-chain-aware authority,
- recovery-aware control,
- and optional large-model fingerprint bindings.

These properties make the method suitable for identity systems in which an intelligent agent must remain independently identifiable, controllable, and interoperable across services, addresses, networks, and AI-related digital environments.
