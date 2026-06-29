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

| Resource form | `resourceType` | DID subject code |
| --- | --- | --- |
| Agent Service | `agent_service` | `AG` |
| Skill | `skill` | `SK` |
| MCP Server | `mcp_server` | `MC` |
| Tool / API | `tool_api` | `TL` |

The DID subject code and `oanMetadata.resourceType` must match. For example,
`did:oan:SKFI:...` describes a `skill`, not an MCP server.

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
Servers, and Tool / API resources should use the matching DID subject code,
resource type, service type, endpoint, and protocol binding.

```json
{
  "@context": ["https://www.w3.org/ns/did/v1"],
  "id": "did:oan:SKFI:REPLACE_WITH_32_CHAR_SUFFIX",
  "controller": "did:oan:ORFI:REPLACE_WITH_CONTROLLER_SUFFIX",
  "verificationMethod": [
    {
      "id": "did:oan:SKFI:REPLACE_WITH_32_CHAR_SUFFIX#key-1",
      "type": "Ed25519VerificationKey2020",
      "controller": "did:oan:SKFI:REPLACE_WITH_32_CHAR_SUFFIX",
      "publicKeyMultibase": "REPLACE_WITH_PUBLIC_KEY"
    }
  ],
  "authentication": [
    "did:oan:SKFI:REPLACE_WITH_32_CHAR_SUFFIX#key-1"
  ],
  "assertionMethod": [
    "did:oan:SKFI:REPLACE_WITH_32_CHAR_SUFFIX#key-1"
  ],
  "service": [
    {
      "id": "did:oan:SKFI:REPLACE_WITH_32_CHAR_SUFFIX#manifest",
      "type": "OANSkillManifest",
      "serviceEndpoint": "https://example.org/path/to/skill.json",
      "version": "1.0.0"
    }
  ],
  "oanMetadata": {
    "subjectType": "skill",
    "resourceType": "skill",
    "publisherDid": "did:oan:ORFI:REPLACE_WITH_CONTROLLER_SUFFIX",
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
  }
}
```

## Registration Flow

Prepare and validate the DID Document before submitting it to a Registrar:

- `resourceDid` starts with `did:oan:`;
- DID subject code matches `resourceType`;
- DID Document `id` equals `resourceDid`;
- `oanMetadata.resourceType` equals the submitted `resourceType`;
- `oanMetadata.authorizedDomains` is present, valid, and covered by the target
  Registrar;
- external artifacts have declared hashes;
- version fields are explicit.

The resource-oriented Registrar API accepts registration submissions through:

```http
POST /resources/register
Content-Type: application/json
```

The request body should include the resource DID, resource type, DID Document,
metadata, and signature required by the target Registrar.

When the request metadata also carries `authorizedDomains`, it must match the
DID Document's `oanMetadata.authorizedDomains`.

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
- DID subject code matches `resourceType`;
- DID Document `id` equals `resourceDid`;
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
