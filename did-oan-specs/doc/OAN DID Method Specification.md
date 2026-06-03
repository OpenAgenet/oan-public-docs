<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

# `did:oan` Method Specification

Version 1.1.0

## Status of This Document

This document specifies the `did:oan` DID method. It is intended to be read
together with the [W3C DID Core Recommendation](https://www.w3.org/TR/did-core/).

The latest version of this document is available from the
[OAN public documents repository](https://github.com/OpenAgenet/oan-public-docs/tree/main/did-oan-specs).

## 1. Introduction

`did:oan` is a decentralized identifier method for OpenAgenet (OAN). It is
designed to identify, describe, discover, distribute, and authenticate resources
in an Agent ecosystem.

The method is not limited to a single business Agent. A `did:oan` DID subject
MAY represent:

- an Agent Service;
- a Skill;
- an MCP Server;
- a Tool / API;
- an OAN infrastructure node such as Root, Registrar, Discovery, or a
  third-party VC issuer;
- or another OAN-governed resource type defined by a future OAN profile.

The method preserves the agent-native properties of the earlier OAN design while
expanding the subject model so that OAN can support multiple Agent-product
forms through one identity and trust layer.

In OAN, a DID Document is not only a key document. It is also the canonical
identity-facing surface for basic semantic description, discovery entry points,
service bindings, controller relationships, and verification bootstrap. More
complete payloads MAY be distributed as Root-verified packages, manifests, or
service-hosted files, but the DID Document SHOULD contain enough semantic
information for Discovery nodes to index the resource and match user needs.

## 2. Design Goals

The `did:oan` method is designed to satisfy the following goals:

1. Identify OAN Agent ecosystem resources in a decentralized and globally unique
   way.
2. Support Agent Service, Skill, MCP Server, Tool / API, and infrastructure-node
   subjects in one DID method.
3. Support authentication, assertion, recovery, and delegated control
   relationships.
4. Support semantic discovery directly from DID Documents through structured
   descriptions, capability tags, examples, and service bindings.
5. Support service, manifest, download, and invocation endpoint discovery.
6. Support address identification across chains, domains, accounts,
   communication systems, or deployment environments.
7. Support optional large-model fingerprint bindings for AI-related subjects.
8. Support OAN-specific lifecycle, governance, and authorization semantics while
   remaining compatible with DID Core processing.
9. Support national-cryptography-compatible deployments through the OAN SM2/SM3
   profile while allowing Ed25519 deployments.

## 3. Why a New DID Method Is Necessary

`did:oan` is introduced because general-purpose DID methods do not, by
themselves, require or standardize several semantics that are central to OAN:

1. **Agent ecosystem resource identity**
   OAN must identify not only Agents, but also Skills, MCP Servers, Tool APIs,
   and infrastructure nodes that participate in trusted Agent interconnection.

2. **Native semantic discovery**
   OAN Discovery nodes need structured descriptions, capability tags, use-case
   examples, and protocol bindings in a consistent DID Document shape.

3. **Protocol-agnostic interaction support**
   OAN resources may use MCP, A2A, AIP, HTTP APIs, RPC, or other interaction
   protocols. The DID method should identify and authenticate the resource
   without replacing those protocols.

4. **Delegation and controller semantics**
   A resource may be controlled by a person, organization, developer team,
   Agent operator, or infrastructure operator. The DID Document must support
   controller keys, delegation chains, recovery authority, and VC-backed
   authorization evidence.

5. **Service and address bindings**
   OAN resources need service, routing, manifest, download, account, domain, and
   network bindings that can be verified and indexed.

6. **Lifecycle and governance awareness**
   OAN Root, bulletin, and Discovery services need a method-level place for
   lifecycle hints, authorization state references, package hashes, and policy
   references.

If these properties were left entirely to application-specific conventions,
different OAN implementations could interpret them inconsistently.

## 4. Conformance

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** in this document are to be interpreted as described in
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174).

This specification conforms to the W3C DID Core data model and terminology.
Where this method introduces additional properties, those properties are
method-specific extensions and MUST NOT invalidate DID Core processing
expectations.

For the purposes of DID Core conformance, this specification defines:

- the method name;
- the method-specific identifier syntax;
- identifier generation and uniqueness expectations;
- DID Document requirements and method-specific extensions;
- creation, resolution, update, recovery, suspension, authorization-revocation,
  and deactivation operations;
- DID resolution output expectations;
- and security and privacy considerations for the method.

## 5. Representations

The primary DID Document representation defined by this specification is
JSON-LD.

When a `did:oan` DID Document is represented as JSON-LD:

- it MUST include `@context`;
- it MUST include `id`;
- and the `@context` value MUST include `https://www.w3.org/ns/did/v1`.

The OAN method context SHOULD be included when OAN-specific properties are used:

```json
"https://w3id.org/oan/v1"
```

Implementations MAY support additional compatible representations as long as
DID Core processing expectations remain satisfied.

OAN-specific properties such as `oanMetadata`, `resourceDescription`,
`protocolBindings`, `implementationLinks`, `addressBindings`,
`delegationChain`, `credentialRequirements`, `packageInfo`, and
`modelFingerprints` are method extensions. JSON-LD processors MUST either use
the OAN method context or otherwise process these properties in a way that does
not conflict with DID Core terms.

## 6. Method Name

The name string that identifies this DID method is:

```text
oan
```

A DID produced by this method MUST begin with the lowercase prefix:

```text
did:oan
```

## 7. System Applicability

`did:oan` is intended for Agent ecosystem systems, trusted Agent
interconnection, and cross-domain digital systems where:

- Agent Services, Skills, MCP Servers, and Tool APIs need stable identity;
- semantic discovery is performed over capability descriptions and tags;
- service endpoints, manifest endpoints, and download endpoints must be
  discovered from verified data;
- multiple public keys or controllers may be associated with one resource;
- requesters and providers may mutually verify DID material, VC evidence, and
  signatures before download or invocation;
- and OAN infrastructure nodes need verifiable lifecycle and authorization
  state.

## 8. `did:oan` Identifier Syntax

### 8.1 General Structure

The method-specific identifier consists of two parts:

1. a four-character semantic code; and
2. a primary identifier suffix.

The four-character semantic code is structured as:

- the first two characters indicate the subject category; and
- the last two characters indicate the application domain.

The general DID form is:

```text
did:oan:<semantic-code>:<suffix>
```

Example:

```text
did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz
```

In the example:

- `AG` indicates the subject category; and
- `FI` is an example application-domain code.

### 8.2 Subject Category Codes

The following subject category codes are defined by this specification:

| Code | Subject category | Recommended `subjectType` |
| --- | --- | --- |
| `AG` | Agent Service or Agent subject | `agent_service` |
| `SK` | Skill | `skill` |
| `MC` | MCP Server | `mcp_server` |
| `TL` | Tool / API | `tool_api` |
| `IN` | OAN infrastructure node | `infrastructure_node` |
| `OR` | Organization or operator subject | `organization` |
| `DV` | Developer or individual operator subject | `developer` |
| `RS` | Reserved resource subject | deployment-defined |

Deployments MAY define additional subject category codes through OAN governance
or registry policy. A deployment-defined code MUST NOT change the meaning of the
codes above.

### 8.3 ABNF

```abnf
oan-did = "did:oan:" oan-specific-identifier
oan-specific-identifier = semantic-code ":" suffix
semantic-code = subject-code app-domain-code
subject-code = 2(UPPER / DIGIT)
app-domain-code = 2(UPPER / DIGIT)
suffix = 32base58-char
base58-char = DIGIT-NONZERO / %x41-48 / %x4A-4E / %x50-5A / %x61-6B / %x6D-7A
UPPER = %x41-5A
DIGIT-NONZERO = %x31-39
```

Interpretation:

- `semantic-code` is a four-character method-level semantic segment.
- `subject-code` identifies the resource or subject category.
- `app-domain-code` is a two-character application-domain code.
- `suffix` is the primary method-specific identifier suffix. It is a fixed
  32-character string using the Bitcoin Base58 alphabet:
  `123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz`.

The `did:oan` method name is lowercase and MUST be serialized as `oan`.
`semantic-code`, `subject-code`, and `app-domain-code` MUST be serialized in
uppercase. Resolvers MAY accept lowercase semantic-code input for usability, but
MUST normalize semantic-code values to uppercase before lookup, comparison, or
storage. The `suffix` is case-sensitive and MUST NOT be case-normalized.

Two `did:oan` identifiers are equivalent only when their canonical serialized
forms are identical after method-name normalization and semantic-code
normalization. Implementations MUST NOT treat two identifiers with different
suffix casing as equivalent.

The canonical textual length of a `did:oan` DID is 45 characters:

```text
8 characters for "did:oan:" + 4 characters for semantic-code + 1 colon + 32 suffix characters
```

### 8.4 Identifier Generation

The OAN identifier generation process is:

1. Select the subject category code and application-domain code.
2. Generate or derive identifier material with at least 160 bits of effective
   collision resistance. The RECOMMENDED generation method is to sample 32
   characters uniformly from the Base58 alphabet using a cryptographically
   secure random number generator and rejection sampling.
3. For deterministic generation, derive a byte stream from a domain-separated
   hash or extendable-output function over stable controller material, such as:

   ```text
   OAN-DID-SUFFIX-v1 || semantic-code || canonical-controller-public-key || optional-nonce
   ```

   The byte stream MUST then be mapped to the Base58 alphabet without modulo
   bias, for example by rejection sampling, until 32 suffix characters are
   produced.
4. Concatenate `did:oan:`, the semantic code, `:`, and the 32-character suffix.
5. Submit the DID for creation or registration. If the canonical DID already
   exists, the creation operation MUST fail and the generator MUST use a new
   nonce or new random material.

The suffix MUST NOT encode the current cryptographic algorithm, service
endpoint, resource name, package hash, or mutable metadata. Cryptographic
algorithm information belongs in DID Core verification methods, and mutable
resource information belongs in the DID Document or DID document metadata.

The DID itself is immutable. Lifecycle changes affect the DID Document and
associated metadata, not the DID string.

The generated DID MUST be globally unique within the `did:oan` method. A
registrar or creation service MUST reject creation if the canonical DID already
exists. When locally generated suffixes are used before registration, the
generator SHOULD use cryptographically strong randomness, a public-key-derived
identifier, or another collision-resistant process approved by the deployment.

## 9. Method-Specific Characteristics

The distinctive characteristics of `did:oan` are:

1. **Resource-centric Agent ecosystem identity**
   The DID subject may be an Agent Service, Skill, MCP Server, Tool / API,
   infrastructure node, or other OAN-governed resource.

2. **Native semantic description**
   DID Documents SHOULD expose structured capability description, tags, examples,
   scenario information, and protocol bindings for discovery.

3. **Service and manifest discovery**
   DID Documents MAY expose invocation endpoints, MCP endpoints, API endpoints,
   manifest endpoints, download endpoints, resolver endpoints, and routing
   endpoints.

4. **Address-aware identity bindings**
   DID Documents MAY expose address information associated with chains,
   networks, accounts, domains, communication endpoints, or service systems.

5. **Delegation-chain awareness**
   DID Documents MAY expose delegated authority relationships and related
   delegation proofs.

6. **Recovery-aware control model**
   The method supports explicit recovery authority distinct from day-to-day
   authentication.

7. **Model-fingerprint-aware bindings**
   DID Documents MAY expose model fingerprint information when the subject is an
   AI system, model-backed Agent Service, or AI-related resource.

8. **OAN lifecycle and authorization awareness**
   DID Documents MAY include lifecycle hints, Root proof references, bulletin
   references, package hashes, and policy hashes. Authoritative lifecycle and
   authorization state is defined by OAN Root and bulletin evidence, not by
   self-asserted hints alone.

## 10. DID Document Requirements

### 10.1 Core DID Requirements

A conforming `did:oan` DID Document MUST satisfy DID Core requirements. In
particular, a valid representation MUST include:

- `@context`
- `id`

The `id` value MUST be the canonical `did:oan` DID that identifies the DID
subject. A DID Document MUST NOT use OAN method-specific metadata to override
the DID Core meaning of `id`, `controller`, `verificationMethod`,
`authentication`, `assertionMethod`, `keyAgreement`, `capabilityInvocation`,
`capabilityDelegation`, `service`, or `alsoKnownAs`.

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

### 10.2 OAN Method-Specific Object

A conforming `did:oan` DID Document SHOULD include a method-specific object
named `oanMetadata`.

The `oanMetadata` object is the primary place for OAN method-specific identity,
resource, semantic description, address, delegation, routing, package,
lifecycle, and governance semantics.

`oanMetadata` is not a DID Core controller object. It MUST NOT be used as a
substitute for DID Core controller relationships, verification relationships, or
service definitions. Where both DID Core properties and `oanMetadata` describe
related information, relying parties MUST use DID Core properties for DID Core
processing and MAY use `oanMetadata` for OAN discovery, policy, and
application-specific interpretation.

### 10.3 `oanMetadata` Properties

The `oanMetadata` object MAY contain:

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| `subjectType` | string | Recommended | Subject type, such as `agent_service`, `skill`, `mcp_server`, `tool_api`, `infrastructure_node`, `organization`, or `developer`. |
| `resourceType` | string | Recommended | Resource type used for OAN discovery and indexing. For the four initial product forms, use `agent_service`, `skill`, `mcp_server`, or `tool_api`. |
| `nodeRole` | string | Optional | OAN infrastructure or Agent role, such as `root`, `registrar`, `discovery`, `vc-issuer`, `service-agent`, `user-agent`, or `test-agent`. |
| `identityType` | string | Optional | Deployment-specific identity classification. |
| `controllerDid` | string | Optional | DID of the subject that controls, operates, or publishes this resource when different from `id` or DID Core `controller`. |
| `publisherDid` | string | Optional | DID of the resource publisher. |
| `issuerDid` | string | Optional | DID of a VC issuer or authorization issuer relevant to the resource. |
| `ttl` | integer | Optional | Resolver or cache hint in seconds. |
| `recovery` | array of string | Optional | Verification method IDs or DID URLs authorized for recovery. |
| `resourceDescription` | object | Recommended | Native semantic resource description used by OAN Discovery. |
| `agentDescription` | object | Optional | Agent-oriented semantic description. For `agent_service` subjects, this MAY mirror or specialize `resourceDescription`. |
| `capabilityTags` | array of string | Optional | OAN capability-domain tags used for semantic discovery and governed filtering. |
| `protocolBindings` | array | Optional | Protocol bindings such as MCP, A2A, AIP, HTTP, RPC, or custom protocols. |
| `implementationLinks` | array | Optional | Links between Skills and implementing Agent Services, MCP Servers, or Tool APIs. |
| `addressBindings` | array | Optional | Address or endpoint identification records. |
| `delegationChain` | array | Optional | Delegation records relevant to control or scoped capability. |
| `credentialRequirements` | array | Optional | VC or credential requirements for download, invocation, update, or session setup. |
| `packageInfo` | object | Optional | Root-verified package, manifest, hash, or download metadata. |
| `modelFingerprints` | array | Optional | Large-model fingerprint bindings for AI-related subjects. |
| `servicePolicy` | string | Optional | Service-discovery, routing, or access policy label. |
| `networkScope` | string | Optional | Network, ecosystem, or domain scope label. |
| `lifecycleState` | string | Optional | Local lifecycle hint such as `draft`, `active`, `suspended`, `revoked`, or `deactivated`. |

`subjectType` and `resourceType` SHOULD be aligned unless a deployment has a
clear reason to distinguish identity category from resource category.

### 10.4 Resource Description

`resourceDescription` is the primary semantic description object for OAN
Discovery. It SHOULD be present for Agent Service, Skill, MCP Server, and
Tool / API subjects.

The object MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `name` | string | Human-readable resource name. |
| `description` | string | Narrative description of the resource. |
| `capabilityDescription` | string | Capability-focused description. |
| `capabilityTags` | array of string | Capability labels or governed OAN capability-domain tags. |
| `useCaseExamples` | array of string | Example scenarios or user requests. |
| `inputSchema` | object or string | Inline input schema or schema reference. |
| `outputSchema` | object or string | Inline output schema or schema reference. |
| `examples` | array | Example prompts, invocations, requests, or outputs. |
| `audience` | string or array | Intended users, systems, or domains. |
| `domain` | string or array | Domain labels such as finance, legal, industrial, healthcare, or education. |
| `language` | string or array | Supported natural languages when relevant. |
| `version` | string | Resource description version. |

The description is intended for discovery, routing, and interoperability. It
MUST NOT be treated as a substitute for cryptographic controller information,
Root proof, package hash verification, VC verification, or executable access
policy.

### 10.5 Subject-Type Profiles

#### 10.5.1 Agent Service

For an Agent Service DID Document:

- `subjectType` SHOULD be `agent_service`;
- `resourceType` SHOULD be `agent_service`;
- `resourceDescription` SHOULD describe the Agent's capabilities;
- `service` SHOULD include one or more invocation, messaging, A2A, AIP, or
  custom Agent service endpoints;
- `protocolBindings` SHOULD describe supported interaction protocols;
- `modelFingerprints` MAY be present for model-backed Agents.

#### 10.5.2 Skill

For a Skill DID Document:

- `subjectType` SHOULD be `skill`;
- `resourceType` SHOULD be `skill`;
- `resourceDescription` SHOULD describe what the Skill can do, its input/output
  expectations, use cases, and examples;
- `packageInfo` or `service` MAY expose a Skill manifest or download endpoint;
- `implementationLinks` SHOULD link the Skill to implementing Agent Services,
  MCP Servers, or Tool APIs when known.

A Skill is a capability resource. It MAY be discovered independently, but actual
execution usually resolves to an implementation resource.

#### 10.5.3 MCP Server

For an MCP Server DID Document:

- `subjectType` SHOULD be `mcp_server`;
- `resourceType` SHOULD be `mcp_server`;
- `service` SHOULD include the MCP endpoint and transport information;
- `protocolBindings` SHOULD include an MCP binding;
- `resourceDescription` SHOULD summarize exposed tools, resources, prompts, and
  use cases.

#### 10.5.4 Tool / API

For a Tool / API DID Document:

- `subjectType` SHOULD be `tool_api`;
- `resourceType` SHOULD be `tool_api`;
- `service` SHOULD include API, RPC, or function endpoint information;
- `resourceDescription` SHOULD describe API purpose, schema, examples, and
  capability tags;
- `credentialRequirements` MAY describe access credentials required before
  invocation.

#### 10.5.5 Infrastructure Node

For an OAN infrastructure node DID Document:

- `subjectType` SHOULD be `infrastructure_node`;
- `nodeRole` SHOULD describe the role, such as `root`, `registrar`,
  `discovery`, or `vc-issuer`;
- `service` SHOULD include operational endpoints where public discovery is
  appropriate;
- `lifecycleState` MAY provide a local hint, but authoritative authorization is
  determined by OAN Root and bulletin evidence.

### 10.6 Protocol Bindings

Each `protocolBindings` entry MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Binding identifier. |
| `protocol` | string | Protocol label, such as `mcp`, `a2a`, `aip`, `https`, `rpc`, or `custom`. |
| `version` | string | Protocol version. |
| `transport` | string | Transport, such as `http`, `sse`, `websocket`, `stdio`, or `grpc`. |
| `serviceRef` | string | DID URL of the related service entry. |
| `schemaRef` | string | Schema, OpenAPI, MCP manifest, or other interface reference. |
| `auth` | string or array | Authentication mode or required OAN credential profile. |
| `summary` | string or object | Human-readable or structured protocol summary. |

### 10.7 Implementation Links

Each `implementationLinks` entry MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Link identifier. |
| `relation` | string | Relation such as `implemented_by`, `exposed_by`, `invokes`, `depends_on`, or `replaces`. |
| `targetDid` | string | DID of the target resource. |
| `targetService` | string | Optional DID URL of a target service. |
| `versionConstraint` | string | Optional version constraint. |
| `description` | string | Optional description of the implementation relation. |

### 10.8 Credential Requirements

Each `credentialRequirements` entry MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Requirement identifier. |
| `purpose` | string | Purpose such as `download`, `invoke`, `update`, `session`, or `publish`. |
| `credentialType` | string | Required VC or credential type. |
| `issuer` | string or array | Accepted issuer DID or issuer set. |
| `scope` | string or array | Required scope or claim. |
| `presentationMode` | string | Presentation style, such as `vp`, `signed-envelope`, or deployment-defined value. |
| `required` | boolean | Whether the credential is mandatory. |

These requirements allow a resource provider to require the requester to present
VC evidence before downloading a Skill file, invoking an MCP Server, calling a
Tool API, or establishing a session. The same model also supports mutual
verification between two Agent Services.

### 10.9 Package Information

`packageInfo` MAY contain Root-verified package or manifest information:

| Property | Type | Description |
| --- | --- | --- |
| `manifestUrl` | string | URL for a manifest or package metadata file. |
| `downloadUrl` | string | URL for a downloadable file or package. |
| `packageHash` | string | Hash of the package. |
| `metadataHash` | string | Hash of the metadata. |
| `rootProofRef` | string | Reference to Root proof material. |
| `bulletinRef` | string | Reference to OAN bulletin event or authorization state. |
| `version` | string | Package version. |

Discovery nodes MAY provide user-facing download endpoints for Root-verified
packages. CDN services are distribution infrastructure and are not trust
authorities.

### 10.10 Address Bindings

Each `addressBindings` entry MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Unique identifier for the binding entry. |
| `addressType` | string | Address class, such as `wallet`, `domain`, `endpoint`, `account`, or deployment-defined type. |
| `network` | string | Network or namespace name. |
| `address` | string | The bound address value. |
| `controller` | string | DID or DID URL asserting control over the address. |
| `purpose` | string | Intended role, such as `payment`, `service`, `routing`, or `identity`. |

### 10.11 Delegation Chain

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

The delegation chain is a method extension for expressing authority routing and
MUST NOT be treated as a substitute for DID Core controller semantics.

### 10.12 Agent Description

`agentDescription` MAY be used by Agent Service subjects for compatibility with
agent-oriented OAN documents. When present, it MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `capabilityDescription` | string | Narrative description of Agent capabilities. |
| `capabilityTags` | array of string | Capability labels or phrases. |
| `useCaseExamples` | array of string | Example application scenarios. |

For new DID Documents, `resourceDescription` is the preferred general semantic
description object. `agentDescription` MAY mirror or specialize it for Agent
Service subjects.

### 10.13 Model Fingerprints

Each `modelFingerprints` entry MAY contain:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Fingerprint entry identifier. |
| `modelProvider` | string | Model provider or operator label. |
| `modelName` | string | Model name. |
| `modelVersion` | string | Model or checkpoint version. |
| `fingerprint` | string | Model fingerprint value. |
| `fingerprintAlgorithm` | string | Fingerprint algorithm identifier. |
| `bindingPurpose` | string | Purpose, such as `agent-identity`, `attestation`, or `service-integrity`. |
| `created` | string | RFC 3339 creation timestamp. |

A model fingerprint is a method-specific identity binding for AI-related
subjects. It MUST NOT be treated as a replacement for controller keys or DID
Core authentication semantics.

### 10.14 Method-Specific `attributes`

`did:oan` MAY include an `attributes` array for application-facing descriptive
metadata.

Each attribute object MAY include:

| Property | Type | Description |
| --- | --- | --- |
| `key` | string | Attribute key. |
| `desc` | string | Human-readable description. |
| `encrypt` | integer | `0` for plaintext, `1` for encrypted or protected. |
| `format` | string | Data type such as `text`, `image`, `video`, `mixture`, or deployment-defined type. |
| `value` | string | Attribute value. |

`attributes` are method extensions for descriptive interoperability and MUST NOT
be treated as a substitute for core controller, resource description, address,
delegation, package, or fingerprint semantics.

### 10.15 Verification Methods

The DID Document SHOULD use DID Core `verificationMethod` rather than the legacy
`publicKey` property.

Supported verification suites MAY include:

- `Ed25519VerificationKey2020`
- `EcdsaSecp256k1VerificationKey2019`
- `SM2VerificationKey2020`

At least one verification method referenced from `authentication` or
`assertionMethod` SHOULD be present unless the DID is permanently deactivated.

OAN implementations SHOULD use the `cryptoSuite` field on verification methods
and proofs when the verification method type alone is not sufficient to select
the signing and hashing suite.

The following crypto suites are defined for OAN compatibility:

| `cryptoSuite` | Verification method type | Proof type | Signing algorithm | Hash algorithm | DID key prefix |
| --- | --- | --- | --- | --- | --- |
| `Ed25519Sha256Legacy` | `Ed25519VerificationKey2020` | `Ed25519Signature2020` | Ed25519 | SHA-256 legacy canonical hash input | `e` |
| `Ed25519Sha256` | `Ed25519VerificationKey2020` | `Ed25519Signature2020` | Ed25519 | SHA-256 | `e` |
| `Sm2Sm3` | `SM2VerificationKey2020` | `SM2Signature2020` | SM2 | SM3 | `z` |

The `Sm2Sm3` suite is the OAN profile for SM2 signing with SM3 hashing. A
verifier that supports `Sm2Sm3` MUST verify both the `cryptoSuite` value and the
verification method key material before accepting a signature.

Where `publicKeyJwk` is used for SM2, the JWK SHOULD identify the key type and
curve using implementation-compatible values such as `kty: "EC"` and
`crv: "SM2"`. Where `publicKeyMultibase` is used, the key encoding profile MUST
be sufficient for the verifier to reconstruct the SM2 public key.

Support for `Sm2Sm3` is part of the OAN method profile because OAN deployments
may need national-cryptography-compatible signing and verification.

### 10.16 Services

The DID Document MAY include service endpoints. For OAN deployments, service
entries commonly include:

- DID resolver or sub-resolver endpoints;
- Agent invocation endpoints;
- MCP Server endpoints;
- Tool / API endpoints;
- Skill manifest or download endpoints;
- messaging or communication endpoints;
- address-resolution services;
- package or metadata endpoints.

Recommended OAN service type labels include:

| Service type | Purpose |
| --- | --- |
| `OANAgentService` | Agent Service invocation, messaging, or task endpoint. |
| `OANMCPServer` | MCP Server endpoint. |
| `OANToolAPI` | Tool or API endpoint. |
| `OANSkillManifest` | Skill manifest or description endpoint. |
| `OANResourcePackage` | Root-verified package or metadata endpoint. |
| `OANDiscoveryDownload` | Discovery-hosted download endpoint. |
| `DIDSubResolver` | DID sub-resolution or routing endpoint. |

If a service is intended to represent a resolution, routing, download, or
invocation service, it SHOULD include:

| Property | Type | Description |
| --- | --- | --- |
| `id` | string | Service identifier. |
| `type` | string | Service type label. |
| `serviceEndpoint` | string or object | Service endpoint. |
| `version` | string | Optional service version. |
| `protocol` | string or integer | Optional transport or protocol indicator. |
| `serverType` | string or integer | Optional deployment/server type indicator. |
| `port` | integer | Optional port value. |

## 11. DID Document Metadata and DID Resolution Metadata

In DID Core, some information belongs in DID document metadata or DID resolution
metadata rather than in the DID Document body itself.

Accordingly, for `did:oan`:

- `created` SHOULD normally be expressed as DID document metadata;
- `updated` SHOULD normally be expressed as DID document metadata;
- `deactivated` MUST be expressed in DID document metadata when relevant;
- resolver status, processing status, or representation-level diagnostics
  SHOULD be expressed in DID resolution metadata.

This specification does not require top-level `created`, `updated`, or a
top-level `proof` object as universal in-document properties, even though
deployments MAY include timestamps or proof material where appropriate.

## 12. State and Control Model

`did:oan` introduces method-specific control-state expectations.

Recognized states MAY include:

- `active`
- `suspended`
- `revoked`
- `deactivated`

Control semantics:

1. **active**
   The DID is active and can be resolved with current controller and service
   information.

2. **suspended**
   The DID or resource is temporarily suspended by policy or governance. This
   state is recoverable.

3. **revoked**
   The DID or resource authorization is revoked. A revoked resource SHOULD NOT
   be accepted for new trusted interactions unless a new DID or new
   authorization record is created.

4. **deactivated**
   The DID has been deactivated according to method rules. Deactivation does not
   require historical erasure.

In addition, the method distinguishes among:

- active authentication authority;
- assertion authority;
- delegated authority;
- recovery authority;
- and resource operation authority.

The subject behind a resource may hold private keys and VC evidence used for
registration, update, download-time verification, or call-time mutual
verification. The execution party may be an Agent, an MCP Server, an API
service, or other server-side logic.

## 13. Method Operations

`did:oan` supports the standard DID lifecycle concepts of creation, update, and
deactivation, but specializes them for OAN resource identities.

The standard DID Method operations are:

- Create
- Read
- Update
- Deactivate

The method also defines method-specific operations such as:

- Recovery
- Delegate Capability
- Revoke Delegation
- Suspend
- Revoke Authorization

### 13.1 Create

Creation establishes a new DID record.

At creation time:

- the DID MUST be unique;
- the method-specific identifier MUST conform to Section 8;
- the DID Document MUST contain the DID subject in `id`;
- `oanMetadata.subjectType` and `oanMetadata.resourceType` SHOULD be set;
- semantic discovery information SHOULD be present for discoverable resources;
- and required signatures or controller authorizations MUST validate.

If recovery authority is defined at creation time, it SHOULD be distinguishable
from normal authentication authority.

Creation authorization MUST be defined by the applicable OAN registrar, Root, or
deployment governance policy. A creation request MUST be rejected if it cannot
prove control over the initial controller material or satisfy the creation
authority required by the deployment.

### 13.2 Read (Resolve)

For the `did:oan` method, the DID Method Read operation is realized through DID
resolution.

Resolution returns a DID Document and DID document metadata.

Resolvers for `did:oan` SHOULD be capable of returning enough information for a
relying party to:

- determine the active controller keys;
- inspect recovery and delegation semantics;
- discover service, manifest, download, or invocation endpoints;
- identify associated addresses;
- inspect semantic resource-description metadata;
- inspect protocol bindings and credential requirements;
- and inspect model-fingerprint bindings relevant to the subject.

### 13.3 Update

Update modifies a DID record while preserving DID identity.

For `did:oan`, updates MAY include changes to:

- verification methods;
- controller references;
- service endpoints;
- resource description;
- protocol bindings;
- package information;
- recovery references;
- address bindings;
- delegation records;
- credential requirements;
- lifecycle hints;
- and model-fingerprint metadata.

Implementations SHOULD clearly distinguish between normal updates,
recovery-driven updates, delegation-related updates, and governance-driven
lifecycle changes.

An update MUST be authorized by the current controller, a valid capability
invocation authority, a valid recovery authority, or another governance
authority recognized by the deployment. Updates that change controller material,
service endpoints, package hashes, authorization references, or recovery
authority SHOULD be protected by stronger authorization policy than updates to
non-security-critical descriptive metadata.

### 13.4 Recovery

Recovery replaces or restores control according to method recovery policy.

Recovery is especially relevant when:

- controller keys are lost;
- controller keys are compromised;
- operational ownership changes;
- or governance policy requires controlled key replacement.

Recovery MUST be authorized by a valid recovery authority as defined by the DID
Document or method rules.

### 13.5 Delegate Capability

`Delegate Capability` is a method-specific operation of `did:oan`.

It is used when a controller authorizes another DID, DID URL, or service-bound
identity to act within a defined scope.

A delegation operation SHOULD define:

- the delegator;
- the delegate;
- the delegated capability;
- the scope;
- the validity period;
- and any attached delegation proof.

### 13.6 Revoke Delegation

`Revoke Delegation` is used when previously granted delegated authority must be
withdrawn.

Implementations SHOULD preserve a verifiable record that the delegation existed
and was later revoked, where legally and operationally appropriate.

### 13.7 Suspend

`Suspend` marks the DID subject or resource as temporarily not eligible for
trusted use.

For OAN infrastructure or governed resource subjects, authoritative suspension
SHOULD be reflected by OAN Root and bulletin evidence. DID Document
`lifecycleState` is only a local hint unless supported by authoritative
evidence.

### 13.8 Revoke Authorization

`Revoke Authorization` withdraws authorization for a governed resource or
infrastructure subject.

The DID Document MAY remain resolvable for historical verification, but relying
parties SHOULD reject new trusted use if current OAN authorization state is
revoked.

### 13.9 Deactivate

Deactivation marks the DID as no longer active.

For `did:oan`, deactivation:

- MUST NOT require historical erasure;
- SHOULD preserve verifiability of past controller and delegation facts where
  legally and technically possible;
- and SHOULD be reflected in DID document metadata using `deactivated: true`.

Deactivation MUST be authorized by the current controller, a valid recovery
authority, or a governance authority recognized by the deployment. Once
deactivated, a DID MUST NOT be reactivated as the same active subject unless the
deployment explicitly defines a recovery process that preserves the historical
deactivation evidence and prevents relying-party confusion.

## 14. DID Resolution

### 14.1 Resolution Output

Resolvers for `did:oan` MUST return outputs consistent with DID Core.

A successful DID resolution result MUST include:

- DID resolution metadata;
- a DID Document, unless the DID is not found, not resolvable, or a requested
  representation cannot be produced;
- and DID document metadata.

When a DID is deactivated, the resolver SHOULD return DID document metadata with
`deactivated: true`. The resolver MAY return the last known DID Document or an
empty DID Document according to deployment policy, but relying parties MUST
treat the DID as inactive for new trusted use.

### 14.2 DID Document Metadata

For `did:oan`, DID document metadata MAY include:

| Property | Type | Description |
| --- | --- | --- |
| `created` | string | DID creation time. |
| `updated` | string | Last update time. |
| `deactivated` | boolean | Whether the DID is deactivated. |
| `controllerState` | string | Current control state. |
| `networkScope` | string | Current network or domain scope. |
| `resolvedAddresses` | array | Resolved address bindings, if exposed by the resolver. |
| `authorizationState` | string | Current OAN authorization state if known to the resolver. |
| `packageState` | string | Current package or publication state if known. |

### 14.3 Resolution Semantics

If the DID identifies a subject with service, resource-description, address,
delegation, package, or model-fingerprint semantics, the resolver SHOULD return
enough information for an external verifier to:

1. determine current controller and authentication methods;
2. inspect recovery authority;
3. inspect available delegation-chain information;
4. discover service, manifest, download, and invocation endpoints;
5. inspect associated address bindings;
6. inspect semantic description and capability tags;
7. inspect protocol bindings and credential requirements;
8. inspect package and Root proof references;
9. and inspect any model-fingerprint bindings relevant to the subject.

Resolvers SHOULD make the trust basis of their responses clear. Where possible,
resolution responses SHOULD expose or reference the registry state, Root proof,
bulletin proof, package hash, signature, or other evidence used to determine
the current DID Document and DID document metadata. Relying parties SHOULD NOT
assume that an unauthenticated resolver response is authoritative merely because
it is syntactically valid.

## 15. Discovery and Distribution Semantics

OAN Discovery nodes SHOULD index DID Documents using:

- `oanMetadata.subjectType`;
- `oanMetadata.resourceType`;
- `oanMetadata.resourceDescription`;
- `oanMetadata.capabilityTags`;
- `oanMetadata.protocolBindings`;
- `service`;
- `oanMetadata.packageInfo`;
- and deployment-approved `attributes`.

For Agent Service, MCP Server, and Tool / API subjects, discovery normally
returns a callable resource candidate. For Skill subjects, discovery normally
returns a capability candidate that may require downloading a Skill package or
resolving implementation links before invocation.

Discovery nodes MAY provide user-facing file downloads for Root-verified Skill
files or resource packages. Relying parties MUST verify Root proof, package
hashes, publisher signatures, and relevant authorization state before trusted
use.

## 16. DID Document Examples

### 16.1 Agent Service Example

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/oan/v1"
  ],
  "id": "did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz",
  "controller": "did:oan:ORFI:4FvNq8Zp2XcR6tYa3Mb7Ws9DhK5GjLeU",
  "verificationMethod": [
    {
      "id": "did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz#key-1",
      "type": "Ed25519VerificationKey2020",
      "controller": "did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz",
      "cryptoSuite": "Ed25519Sha256",
      "publicKeyMultibase": "z6Mkexample"
    },
    {
      "id": "did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz#sm2-1",
      "type": "SM2VerificationKey2020",
      "controller": "did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz",
      "cryptoSuite": "Sm2Sm3",
      "publicKeyJwk": {
        "kty": "EC",
        "crv": "SM2",
        "x": "example-sm2-x-coordinate",
        "y": "example-sm2-y-coordinate"
      }
    }
  ],
  "authentication": [
    "did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz#key-1"
  ],
  "assertionMethod": [
    "did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz#key-1"
  ],
  "service": [
    {
      "id": "did:oan:AGFI:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz#agent-endpoint",
      "type": "OANAgentService",
      "serviceEndpoint": "https://agent.example.org/api",
      "protocol": "a2a",
      "version": "1.0.0"
    }
  ],
  "oanMetadata": {
    "subjectType": "agent_service",
    "resourceType": "agent_service",
    "publisherDid": "did:oan:ORFI:4FvNq8Zp2XcR6tYa3Mb7Ws9DhK5GjLeU",
    "resourceDescription": {
      "name": "Financial Analysis Agent",
      "description": "A professional Agent Service for financial report analysis and risk summarization.",
      "capabilityTags": [
        "finance.report.analysis",
        "finance.risk.summary"
      ],
      "useCaseExamples": [
        "Analyze listed company annual reports.",
        "Summarize market risk signals for a due-diligence workflow."
      ]
    },
    "protocolBindings": [
      {
        "id": "binding-a2a",
        "protocol": "a2a",
        "version": "1.0.0",
        "transport": "http",
        "serviceRef": "#agent-endpoint",
        "auth": ["signed-oan-envelope", "vc-presentation"]
      }
    ],
    "modelFingerprints": [
      {
        "id": "#model-1",
        "modelProvider": "ExampleAI",
        "modelName": "example-model",
        "modelVersion": "2026-04",
        "fingerprint": "sha256:abcdef0123456789",
        "fingerprintAlgorithm": "sha-256",
        "bindingPurpose": "agent-identity",
        "created": "2026-04-29T10:00:00Z"
      }
    ],
    "lifecycleState": "active"
  }
}
```

### 16.2 Skill Example

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/oan/v1"
  ],
  "id": "did:oan:SKLG:5HkPq7Vm3RdT9Ya2WcX8Ns4Bf6GjLeZu",
  "controller": "did:oan:ORLG:8LcR3Vn5YpQw2Tx7Mb9Zd4Fa6GhKsEuJ",
  "verificationMethod": [
    {
      "id": "did:oan:SKLG:5HkPq7Vm3RdT9Ya2WcX8Ns4Bf6GjLeZu#key-1",
      "type": "Ed25519VerificationKey2020",
      "controller": "did:oan:SKLG:5HkPq7Vm3RdT9Ya2WcX8Ns4Bf6GjLeZu",
      "cryptoSuite": "Ed25519Sha256",
      "publicKeyMultibase": "z6Mkskillexample"
    }
  ],
  "authentication": [
    "did:oan:SKLG:5HkPq7Vm3RdT9Ya2WcX8Ns4Bf6GjLeZu#key-1"
  ],
  "service": [
    {
      "id": "did:oan:SKLG:5HkPq7Vm3RdT9Ya2WcX8Ns4Bf6GjLeZu#manifest",
      "type": "OANSkillManifest",
      "serviceEndpoint": "https://discovery.example.org/packages/skills/contract-review.json",
      "protocol": "https",
      "version": "1.0.0"
    }
  ],
  "oanMetadata": {
    "subjectType": "skill",
    "resourceType": "skill",
    "publisherDid": "did:oan:ORLG:8LcR3Vn5YpQw2Tx7Mb9Zd4Fa6GhKsEuJ",
    "resourceDescription": {
      "name": "Contract Review Skill",
      "description": "A reusable Skill for reviewing contracts, extracting clauses, and identifying legal risk signals.",
      "capabilityTags": [
        "legal.contract.review",
        "legal.risk.identification"
      ],
      "inputSchema": "https://example.org/schemas/contract-review-input.json",
      "outputSchema": "https://example.org/schemas/contract-review-output.json",
      "useCaseExamples": [
        "Review a sales contract and highlight termination clauses.",
        "Identify high-risk liability clauses in a draft agreement."
      ]
    },
    "implementationLinks": [
      {
        "relation": "implemented_by",
        "targetDid": "did:oan:AGLG:9MaT4Xq6VnRb2Yp7Wc3Zd8Ef5GjKsHuL",
        "targetService": "did:oan:AGLG:9MaT4Xq6VnRb2Yp7Wc3Zd8Ef5GjKsHuL#agent-endpoint"
      }
    ],
    "packageInfo": {
      "manifestUrl": "https://discovery.example.org/packages/skills/contract-review.json",
      "downloadUrl": "https://discovery.example.org/download/skills/contract-review-1.0.0.zip",
      "packageHash": "sha256:1234567890abcdef",
      "version": "1.0.0"
    }
  }
}
```

### 16.3 MCP Server Example

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/oan/v1"
  ],
  "id": "did:oan:MCLG:3NqV7Yp5TxRb9Wc2Md6Za4Ef8GhKsJuL",
  "controller": "did:oan:ORLG:8LcR3Vn5YpQw2Tx7Mb9Zd4Fa6GhKsEuJ",
  "service": [
    {
      "id": "did:oan:MCLG:3NqV7Yp5TxRb9Wc2Md6Za4Ef8GhKsJuL#mcp",
      "type": "OANMCPServer",
      "serviceEndpoint": "https://mcp.example.org/legal",
      "protocol": "mcp",
      "version": "2025-06"
    }
  ],
  "oanMetadata": {
    "subjectType": "mcp_server",
    "resourceType": "mcp_server",
    "resourceDescription": {
      "name": "Legal Tools MCP Server",
      "description": "An MCP Server exposing legal search and clause-analysis tools.",
      "capabilityTags": [
        "legal.search",
        "legal.clause.analysis"
      ],
      "useCaseExamples": [
        "Search legal precedents.",
        "Analyze a contract clause through MCP tools."
      ]
    },
    "protocolBindings": [
      {
        "protocol": "mcp",
        "version": "2025-06",
        "transport": "http",
        "serviceRef": "#mcp",
        "summary": {
          "tools": ["case_search", "clause_analyzer"]
        },
        "auth": ["vc-presentation"]
      }
    ]
  }
}
```

### 16.4 Tool / API Example

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/oan/v1"
  ],
  "id": "did:oan:TLFI:7BcD3Fg5HjK8Mn9Pq2Rs4Tv6WxYzA1Ee",
  "controller": "did:oan:ORFI:2QwR6Ty8VpXc3Mb7Nz4Fd9Gh5KsJeLuA",
  "service": [
    {
      "id": "did:oan:TLFI:7BcD3Fg5HjK8Mn9Pq2Rs4Tv6WxYzA1Ee#api",
      "type": "OANToolAPI",
      "serviceEndpoint": "https://api.example.org/risk-score",
      "protocol": "https",
      "version": "1.0.0"
    }
  ],
  "oanMetadata": {
    "subjectType": "tool_api",
    "resourceType": "tool_api",
    "resourceDescription": {
      "name": "Enterprise Risk Score API",
      "description": "A callable API that calculates enterprise risk scores from structured company data.",
      "capabilityTags": [
        "finance.risk.score",
        "enterprise.analysis"
      ],
      "inputSchema": "https://example.org/schemas/risk-input.json",
      "outputSchema": "https://example.org/schemas/risk-output.json",
      "useCaseExamples": [
        "Score a supplier before procurement onboarding.",
        "Evaluate enterprise risk as part of a due-diligence workflow."
      ]
    },
    "credentialRequirements": [
      {
        "purpose": "invoke",
        "credentialType": "OANServiceAccessCredential",
        "issuer": "did:oan:INFI:6JrTn2Qw8VcY5LpM9XzA3Bd4Ef7GhKsU",
        "required": true
      }
    ]
  }
}
```

## 17. DID Method Compliance Notes

This specification is aligned with DID Core and keeps `did:oan` focused on
method-level semantics.

In particular:

- DID Core `verificationMethod` is used instead of legacy key-description
  conventions;
- transport-specific HTTP request and response payloads are not standardized
  because DID Core does not require a DID method to define a single transport
  API;
- OAN-specific semantic fields are method extensions and do not replace DID Core
  controller, authentication, assertion, or service semantics;
- resource descriptions are used for discovery and interoperability, not as
  executable policy;
- and fields are classified as required, optional, metadata-level, or
  deployment-specific according to DID Core processing expectations.

## 18. Security Considerations

### 18.1 Controller Compromise

If a controller key is compromised, an attacker may attempt unauthorized
updates, service redirection, package replacement, or delegation abuse.
Implementations SHOULD support recovery and SHOULD separate authentication from
longer-term recovery authority where appropriate.

Implementations SHOULD support key rotation and SHOULD record enough update
history or evidence for relying parties to distinguish legitimate rotation from
unauthorized controller replacement.

### 18.2 Resource Misrepresentation

A DID subject may falsely describe its capabilities, endpoints, schemas, or
examples. Relying parties SHOULD verify Root proof, package hashes, publisher
signatures, VC evidence, service authorization, and runtime behavior where
appropriate.

### 18.3 Delegation Abuse

Delegation relationships SHOULD be scoped, time-bounded where appropriate, and
revocable. Implementations SHOULD ensure that delegated capabilities do not
silently exceed the intended scope.

### 18.4 Resolver Trust

Resolvers SHOULD NOT be treated as the sole source of truth. Relying parties
SHOULD verify controller keys, delegation evidence, Root proof references,
package hashes, and important metadata where possible.

Resolver operators SHOULD protect against cache poisoning, stale-state replay,
incorrect deactivation status, downgrade of verification material, and
unauthorized substitution of service endpoints or package references.

### 18.5 Address and Service Misbinding

Because `did:oan` can expose service and address bindings, implementations
SHOULD protect against stale, malicious, or unauthorized endpoint reassignment.

Service endpoints can reveal network locations and can redirect relying parties
to attacker-controlled systems if improperly updated. Relying parties SHOULD
verify that service endpoints are bound to the DID subject through controller
authorization, VC evidence, package proof, or deployment governance evidence
where appropriate.

### 18.6 Model-Fingerprint Misrepresentation

Where model fingerprints are used, implementations SHOULD clarify the
provenance, scope, and meaning of the fingerprint and SHOULD NOT imply stronger
assurance than the fingerprinting method actually provides.

### 18.7 Download and Invocation Authorization

For downloadable Skills or callable services, the provider MAY require the
requester to present VC evidence or signed OAN envelopes before download,
invocation, or session setup. Requesters SHOULD also verify provider DID
Documents, Root proof, package hash, endpoint binding, and current authorization
state before trusted use.

### 18.8 Identifier Confusion and Normalization

Because the method-specific identifier contains both semantic-code and
case-sensitive suffix material, implementations MUST apply the normalization
rules in Section 8 consistently. Inconsistent case normalization can cause two
different DIDs to be treated as the same identifier, or one DID to be
unresolvable in some resolvers.

### 18.9 Extension Property Misuse

OAN method-specific properties are intended to support discovery, governance,
and interoperability. They MUST NOT be interpreted as executable policy unless
the relying party has explicitly adopted the relevant OAN profile. In
particular, descriptive capability metadata MUST NOT be treated as proof that a
resource is safe, authorized, or semantically correct.

## 19. Privacy Considerations

### 19.1 Data Minimization

Identity-related metadata SHOULD be minimized to what is needed for
interoperability and verification. Sensitive personal information SHOULD NOT be
embedded into a DID Document unless strictly necessary.

### 19.2 Public Metadata Awareness

Because DID Documents are often publicly resolvable, method-specific properties
such as `attributes`, `service`, `addressBindings`, `delegationChain`,
`resourceDescription`, `agentDescription`, `modelFingerprints`,
`credentialRequirements`, and `packageInfo` SHOULD be populated with care.

### 19.3 Encrypted or Protected Attributes

Where application-specific metadata is sensitive, implementations MAY use
protected references or encrypted payload strategies and indicate their use
through the `attributes.encrypt` field or service-mediated access control
patterns.

### 19.4 Correlation Risk

Stable DIDs, service endpoints, address bindings, capability tags, package
links, and model fingerprints can make resources, operators, or users easier to
correlate across contexts. Deployments SHOULD avoid placing unnecessary personal
data, account data, private endpoints, or operational topology in public DID
Documents.

### 19.5 Endpoint and Download Privacy

Discovery and download flows can reveal requester interests. Providers and
Discovery nodes SHOULD avoid unnecessary logging, SHOULD use transport security,
and MAY require privacy-preserving authorization flows where sensitive Skills,
Tools, APIs, or Agent Services are involved.

## 20. IANA and Registry Considerations

If `did:oan` is submitted to the W3C DID Method Registry, the registration
SHOULD emphasize that the method is intended for unified identification,
semantic description, trusted discovery, service binding, delegated control,
recovery-aware operations, national-cryptography-compatible verification, and
authenticated use of Agent ecosystem resources.

## 21. Conclusion

`did:oan` is a DID method for OAN Agent ecosystem resources. It supports:

- Agent Service, Skill, MCP Server, Tool / API, and infrastructure-node
  identity;
- semantic description and capability tags for Discovery;
- service, manifest, download, and invocation endpoint discovery;
- controller, delegation, recovery, and VC-backed trust semantics;
- Root-verified package and bulletin-aware lifecycle references;
- optional model-fingerprint bindings;
- and OAN-compatible cryptographic suites including SM2/SM3.

These properties make the method suitable for an identity system in which many
forms of Agent ecosystem products must remain identifiable, controllable,
discoverable, distributable, and verifiable across heterogeneous protocols and
service environments.
