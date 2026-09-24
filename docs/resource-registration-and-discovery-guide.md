<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

# Resource Registration and Discovery Guide

This guide describes how public OAN users model, register, discover, and verify
resources through OAN services.

OAN currently treats these resource forms as first-class registration targets:

- Agent Service
- Skill
- MCP Server
- Tool / API

OAN uses DID Documents as the identity-facing and discovery-facing description
for resources. Discovery indexes Root-approved resource metadata and DID
Documents. It is not a general file host: Skill packages, OpenAPI documents,
MCP manifests, binaries, and other artifacts should be referenced by URL plus
hash.

## Resource Types

Use `did:oan` identifiers for new resources. Use `resourceDid` as the public
resource identifier, and use `oanMetadata.resourceType` to distinguish product
forms.

| Resource form | `subjectType` | `resourceType` |
| --- | --- | --- |
| Agent Service | `agent_instance` or `agent_service` | `agent_service` |
| Skill | `skill` or the applicable owning subject type | `skill` |
| MCP Server | `mcp_server` or the applicable owning subject type | `mcp_server` |
| Tool / API | `tool_api` or the applicable owning subject type | `tool_api` |

The DID string does not encode the resource type. Its five-character
`registrar-code` identifies the initial Registrar source; classification comes
from `oanMetadata.subjectType` and `oanMetadata.resourceType`.

## Metadata To Prepare

Before registering a resource, prepare:

- resource form and `resourceType`;
- human-readable name and description;
- capability tags and use cases;
- authorized domains, such as `legal` or `finance.payments`;
- publisher or controller DID;
- service endpoint, if the resource is callable;
- manifest, package, schema, or download URL, if the resource has an external
  artifact;
- package or metadata hash when an external artifact is referenced;
- version string;
- credential requirements, if download or invocation requires verifiable
  evidence;
- target Registrar and Discovery base URLs.

Do not invent real endpoints, hashes, keys, signatures, credentials, or Root
proofs. Draft values should be clearly marked as placeholders.

`capabilityTags` and `authorizedDomains` are different fields. Capability tags
help users and Discovery find and rank resources. `authorizedDomains` is the
typed authorization-domain list that Registrars, Root, and Discovery use for
admission and publication boundaries. `[]` means no authorization. `["*"]`
means all domains, and is normally reserved for infrastructure nodes rather
than ordinary public resources.

## DID Document Shape

A Skill resource can be modeled with this minimal shape. Agent Services, MCP
Servers, and Tool / API resources should use the applicable `subjectType`,
matching resource type, service type, endpoint, and protocol binding.

```json
{
  "@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/oan/v1"],
  "id": "did:oan:K7mQ9:REPLACE_WITH_32_CHAR_SUFFIX",
  "controller": "did:oan:P9aBc:REPLACE_WITH_CONTROLLER_SUFFIX",
  "verificationMethod": [
    {
      "id": "did:oan:P9aBc:REPLACE_WITH_CONTROLLER_SUFFIX#key-1",
      "type": "Ed25519VerificationKey2020",
      "controller": "did:oan:P9aBc:REPLACE_WITH_CONTROLLER_SUFFIX",
      "publicKeyMultibase": "REPLACE_WITH_PUBLIC_KEY"
    }
  ],
  "authentication": [
    "did:oan:P9aBc:REPLACE_WITH_CONTROLLER_SUFFIX#key-1"
  ],
  "assertionMethod": [
    "did:oan:P9aBc:REPLACE_WITH_CONTROLLER_SUFFIX#key-1"
  ],
  "service": [
    {
      "id": "did:oan:K7mQ9:REPLACE_WITH_32_CHAR_SUFFIX#manifest",
      "type": "OANSkillManifest",
      "serviceEndpoint": "https://example.org/path/to/skill.json",
      "version": "1.0.0"
    }
  ],
  "oanMetadata": {
    "subjectType": "skill",
    "resourceType": "skill",
    "publisherDid": "did:oan:P9aBc:REPLACE_WITH_CONTROLLER_SUFFIX",
    "externalIdentifiers": [
      {
        "id": "urn:example:skill:123",
        "resolutionServiceEndpoint": "https://example.org/resolve"
      }
    ],
    "authorizedDomains": ["example"],
    "resourceDescription": {
      "name": "Example Skill",
      "description": "Describe what the resource does in user-facing terms.",
      "capabilityTags": ["example.skill", "domain.example"],
      "useCases": [
        "Describe a concrete user request this resource can help satisfy."
      ],
      "inputs": ["Describe expected inputs."],
      "outputs": ["Describe expected outputs."],
      "version": "1.0.0"
    },
    "protocolBindings": [
      {
        "protocol": "https",
        "version": "1.0.0",
        "serviceRef": "#manifest",
        "schemaRef": "https://example.org/path/to/schema.json"
      }
    ],
    "packageInfo": {
      "manifestUrl": "https://example.org/path/to/skill.json",
      "downloadUrl": "https://example.org/path/to/skill.zip",
      "packageHash": "sha256:REPLACE_WITH_HASH",
      "metadataHash": "sha256:REPLACE_WITH_METADATA_HASH",
      "hashAlgorithm": "sha256",
      "version": "1.0.0",
      "versionScheme": "semver"
    },
    "credentialRequirements": [
      {
        "purpose": "download",
        "acceptedCredentialTypes": ["OANAccessCredential"],
        "required": false
      }
    ]
  },
  "proof": {
    "type": "Ed25519Signature2020",
    "creator": "did:oan:P9aBc:REPLACE_WITH_CONTROLLER_SUFFIX#key-1",
    "created": "REPLACE_WITH_RFC3339_TIME",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "did:oan:P9aBc:REPLACE_WITH_CONTROLLER_SUFFIX#key-1",
    "cryptoSuite": "ed25519-sha256",
    "hashAlgorithm": "sha256",
    "proofValue": "REPLACE_WITH_PROOF"
  }
}
```

## Registration Flow

Prepare and validate the DID Document before submitting it to a Registrar:

- `resourceDid` matches `did:oan:<5 Base58 registrar-code>:<32 Base58 suffix>`;
- DID Document `id` equals `resourceDid`;
- top-level `controller` is present and is the authoritative control relation;
- `oanMetadata.resourceType` equals the submitted `resourceType`;
- `oanMetadata.authorizedDomains` is present, valid, and covered by the target
  Registrar;
- external artifacts have declared hashes;
- version fields are explicit;
- the top-level proof verifies after removing only the proof field;
- the top-level proof contains `creator`, `verificationMethod`, `cryptoSuite`,
  and `hashAlgorithm`, and `creator` equals `verificationMethod`;
- the submitted DID Document hash equals the hash of the complete document,
  including proof and external identifiers.

The resource-oriented Registrar API accepts registration submissions through:

```http
POST /resources/register
Content-Type: application/json
```

The request body should include the resource DID, resource type, DID Document,
metadata, and signature required by the target Registrar.

When the request metadata also carries `authorizedDomains`, it must match the
DID Document's `oanMetadata.authorizedDomains`.

When the top-level `controller` differs from the submitted resource DID, the
registration request must also include `controllerAuthorizationProof`. The
proof is signed by the controller
identity key and binds the exact resource DID, controller DID, DID Document
hash, metadata hash, Registrar DID, purpose, nonce, expiry, and verification
method. Do not upload controller private key material; the Registrar and Root
need only the public controller DID Document material and the signature proof.
`publisherDid` is descriptive unless it equals the verified top-level controller or
a future publisher-proof extension is introduced.

If the Registrar uses a two-step flow, create or update the draft first, then
submit the completed resource:

```http
POST /resources/submit
Content-Type: application/json
```

After Registrar submission, Root verification and publication make the resource
eligible for downstream publication and Discovery indexing.

## Discovery Flow

Query Discovery with a user need, optional capability tags, and optional
resource type:

```http
POST /discovery/resources/query
Content-Type: application/json
```

Example:

```json
{
  "query": "Find a contract review Skill that extracts clauses and flags legal risk.",
  "resourceType": "skill",
  "capabilityTags": ["legal.contract.review"],
  "limit": 10
}
```

Discovery returns resource candidates with resource identifiers, descriptions,
lifecycle state, version information, service or artifact references, and proof
material when available.

## Verification Checklist

Before downloading or invoking a resource candidate, verify:

- the DID is a valid `did:oan` identifier;
- the DID uses the five-character case-sensitive Registrar code profile;
- `subjectType` and `resourceType` come from the DID Document rather than the
  DID string;
- DID Document `id` equals `resourceDid`;
- the DID Document top-level proof verifies and its complete hash includes the
  proof;
- Root proof binds the resource DID, resource type, version, package hash, and
  metadata hash;
- downloaded artifacts match declared hashes;
- lifecycle state is active;
- credential requirements are understood before download or invocation;
- endpoints and manifest references come from the resolved DID Document or
  Root-approved package metadata.

Semantic description helps users find a resource, but it is not proof of
quality, safety, or correctness. OAN verifies identity, publication,
authorization, and integrity facts.

## Compatibility Notes

Use the current resource-oriented API surface. New integrations should not use
legacy Agent-only routes such as `/agents/draft` or
`/root/agents/verify-and-publish`, and should not create new `did:ans`
identifiers or `ansMetadata` fields.
