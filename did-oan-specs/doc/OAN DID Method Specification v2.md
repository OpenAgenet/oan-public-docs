<!-- Copyright (c) 2026 OpenAgenet contributors -->

# `did:oan` Method Specification — Profile v2

Status: implementation target frozen for the OAN DID profile upgrade.

This specification defines the profile that replaces the legacy four-character
`semantic-code` form. It changes the identifier data model and integrity
envelope while preserving the existing Registrar → Root → CDN → Discovery
business flow, authorization semantics, distribution behavior, and semantic
search behavior.

## 1. Identifier syntax

The canonical form is:

```text
did:oan:<registrar-code>:<resource-suffix>
```

ABNF:

```abnf
oan-did = "did:oan:" registrar-code ":" resource-suffix
registrar-code = 5base58-char
resource-suffix = 32base58-char
base58-char = %x31-39 / %x41-48 / %x4A-4E / %x50-5A /
              %x61-6B / %x6D-7A
```

Both segments are case-sensitive. Implementations MUST NOT uppercase,
lowercase, or otherwise normalize them. The canonical textual length is 46
characters.

Example:

```text
did:oan:K7mQ9:7YpQm9Kx2VnRb6Ts3WfHa4Cd5Ej8LgNz
```

`registrar-code` identifies the initial Registrar source. It is not a resource
type, authorization domain, trust level, lifecycle state, or proof of current
authorization. Resource classification is carried by the DID Document.

## 2. DID Document profile

A production registration document MUST contain:

- DID Core `id`;
- top-level `controller` as one DID or an array of DIDs;
- at least one `verificationMethod`;
- the verification relationship required by the proof;
- `oanMetadata.subjectType`;
- `oanMetadata.resourceType`;
- top-level OAN Data Integrity `proof`.

Top-level `controller` is the authoritative control relationship. New profile
documents MUST NOT rely on `oanMetadata.controllerDid` as an independent
authority. A migration reader that temporarily accepts both fields MUST reject
the document unless they express the same controller relationship.

`nodeRole` is not part of profile-v2 output and MUST NOT be used as an
authorization input. Infrastructure role authorization comes from protected
governance/request context plus DID Document type/service consistency checks.

## 3. OAN metadata types

`subjectType` describes the DID subject. `resourceType` describes how OAN
processes the represented resource. They MUST NOT be inferred from the DID
string.

Frozen values are:

| Value | `subjectType` | `resourceType` |
| --- | --- | --- |
| `agent_instance` | yes | yes |
| `agent_product` | yes | yes |
| `organization` | yes | yes |
| `developer` | yes | yes |
| `agent_service` | yes | yes |
| `skill` | yes | yes |
| `mcp_server` | yes | yes |
| `tool_api` | yes | yes |
| `infrastructure_node` | yes | yes |
| `root_node` | optional | yes |
| `registrar_node` | optional | yes |
| `discovery_node` | optional | yes |
| `cdn_node` | optional | yes |
| `trust_indexer_node` | optional | yes |
| `vc_issuer_node` | optional | yes |
| `unspecified` | yes | yes |

Type fields do not grant authorization and do not replace Root or governance
state.

## 4. External identifiers

External declarations are stored at:

```text
didDocument.oanMetadata.externalIdentifiers
```

Each item contains:

```json
{
  "id": "urn:example:agent:123",
  "resolutionServiceEndpoint": "https://example.org/resolve"
}
```

Rules:

- `id` is required, non-empty, unique in the array, and MUST NOT be a
  `did:oan` identifier;
- `resolutionServiceEndpoint` is optional and, when present, MUST be an
  absolute URI without credentials;
- `file:`, `data:`, and `javascript:` endpoints are forbidden;
- maximum 8 entries;
- maximum `id` length 512 characters;
- maximum endpoint length 1024 characters;
- maximum serialized array size 8192 bytes.

OAN records these values as Controller-signed declarations. Registrar, Root,
CDN, Discovery, SDK, and Trust Indexer MUST NOT treat them as verified external
facts. Core nodes MUST NOT actively access their resolution endpoints.

Registration VCs MAY copy external identifier `id` values but MUST NOT copy
resolution endpoints. External identifiers do not enter governance state or
semantic embedding text.

## 5. Top-level proof and hash order

Profile v2 uses one top-level proof object:

```json
{
  "type": "Ed25519Signature2020",
  "creator": "did:oan:...#key-1",
  "created": "2026-09-25T00:00:00Z",
  "proofPurpose": "assertionMethod",
  "verificationMethod": "did:oan:...#key-1",
  "cryptoSuite": "ed25519-sha256",
  "hashAlgorithm": "sha256",
  "proofValue": "..."
}
```

`creator` and `verificationMethod` MUST identify the same verification method.
`proofPurpose` is `assertionMethod`. The referenced method MUST exist in the
document, be authorized by a current controller, and appear in the required
verification relationship. Proof type, crypto suite, key type, and signature
algorithm MUST match. `type`, `creator`, `created`, `proofPurpose`,
`verificationMethod`, `cryptoSuite`, `hashAlgorithm`, and `proofValue` are all
required for a profile-v2 production registration document.

The mandatory operation order is:

```text
assemble complete DID Document without top-level proof
→ canonical JSON of the document with proof removed
→ Controller signs and creates top-level proof
→ attach proof
→ hash the complete DID Document including proof
→ create controllerAuthorizationProof bound to that final hash
```

The top-level proof does not sign itself, but it is included in the final DID
Document hash. No field may be appended, removed, or changed after proof/hash
generation without regenerating both proof and authorization evidence.

Registrar and Root independently recompute and compare the final hash. CDN
preserves it without rewriting the document. Discovery verifies the package
using the same shared canonical/hash rules.

## 6. Proof responsibility boundaries

| Evidence | Responsibility |
| --- | --- |
| DID Document top-level proof | Controller assertion and document integrity |
| `controllerAuthorizationProof` | Controller authorization for this registration/update operation |
| Registrar registration VC | Registrar attestation that registration completed |
| Root proof | Root acceptance into the OAN publication system |
| Governance state | Current authorization of governed infrastructure DIDs |

None of these replaces another.

## 7. VC and VP minimum profile

OAN recognizes four credential responsibilities:

- Root infrastructure authorization VC;
- Registrar resource registration VC;
- third-party business-fact VC;
- Controller self-claimed capability VC.

The upgrade freezes their responsibility boundaries, common issuer/subject/
proof structure, and structured `credentialStatus` parsing. It does not create
a universal status service, business-fact reviewer, or VP network service.
Static `credentialStatus.status = "active"` is not sufficient evidence of
current validity.

VP support is limited to data structure, proof input construction, and parsing.
VP is not a required input to ordinary resource registration.

## 8. Method operations

### Create

Obtain the target Registrar's Root-registered code, create a 32-character
Base58 suffix, assemble the full document, generate the top-level proof, compute
the final hash, and submit through the existing Registrar and Root flow.

### Resolve

Use existing Root/CDN/Discovery interfaces. A resolved document is not
automatically registered, Root-accepted, active, or governance-authorized.

### Update

Any document content change requires a new top-level proof, final hash, and
operation authorization proof, followed by the existing update/registration
flow.

### Deactivate

Deactivation continues to be expressed by Root resource lifecycle facts or,
for governed infrastructure subjects, existing governance state. The DID
string does not encode active status.

## 9. Governance boundary

Ordinary resource DID recreation does not update on-chain governance.
When Root, Registrar, Discovery, or a governed VC issuer receives a new DID,
the new DID MUST be authorized and the old DID lifecycle handled through the
existing governance process. Existing Trust Indexer logic then indexes those
facts. This profile does not change the governance contract, trust semantics,
or Trust Indexer event interpretation.

## 10. Activation and compatibility

Profile v2 activates only after the shared Rust model, TypeScript SDK, node
validators, fixtures, configuration, and isolated integration tests use the
same frozen rules. A production deployment MUST NOT accept profile-v1 and
profile-v2 as parallel active registration formats.

Legacy profile-v1 materials may remain only as read-only migration evidence or
explicit negative-test fixtures.
