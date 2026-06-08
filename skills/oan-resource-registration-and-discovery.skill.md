<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

---
name: oan-resource-registration-and-discovery
title: OpenAgenet OAN Resource Registration and Discovery Skill
version: 0.1.0
status: draft
audience:
  - AI assistants
  - developer agents
  - integration engineers
  - resource providers
purpose: Help a user prepare, register, discover, and verify OAN resources through OpenAgenet services.
not_for: This Skill itself is not intended to be registered as an OAN discoverable resource.
---

# OpenAgenet OAN Resource Registration and Discovery Skill

## 1. Mission

Use this Skill when a user wants help using OpenAgenet / OAN to register,
publish, discover, or verify a resource.

OAN resources currently include:

- Agent Service
- Skill
- MCP Server
- Tool / API

This Skill is an AI-facing operating guide. It helps an assistant produce OAN
registration materials and call OAN services. It is not itself a resource that
must be published into OAN.

## 2. Operating Principles

When using OAN, treat the DID Document as the identity-facing and
discovery-facing object.

Use `did:oan` only. Do not create `did:ans` identifiers or `ansMetadata` for
new resources.

Use `resourceDid` as the public resource identifier. Do not use `agentDid` as
the primary identifier.

Use `oanMetadata.resourceType` to distinguish product forms:

| Resource form | `resourceType` | DID subject code |
| --- | --- | --- |
| Agent Service | `agent_service` | `AG` |
| Skill | `skill` | `SK` |
| MCP Server | `mcp_server` | `MC` |
| Tool / API | `tool_api` | `TL` |

The DID subject code and `oanMetadata.resourceType` must match. For example,
`did:oan:SKFI:...` must describe a `skill`, not an MCP Server.

OAN Discovery indexes Root-approved DID Documents and resource metadata. It is
not a general file host. Skill files, OpenAPI documents, MCP manifests, code
packages, and other external artifacts should be referenced by URL plus hash.

## 3. Inputs to Collect From the User

Before generating a registration request, collect:

- Resource form: Agent Service, Skill, MCP Server, or Tool / API.
- Human-readable name.
- Description of what the resource does.
- Capability tags.
- Use cases or example user requests.
- Publisher or controller DID, if already available.
- Service endpoint, if the resource is callable.
- Manifest or download URL, if the resource has an external artifact.
- Package or artifact hash, if an external artifact is referenced.
- Version string, such as `1.0.0`, `2026-06`, or another declared scheme.
- Credential requirements, if download or invocation requires VC evidence.
- Target Registrar, Root, and Discovery base URLs.

If required inputs are missing, ask for them or generate a clear draft with
placeholders. Do not invent real endpoints, hashes, keys, or credentials.

## 4. Resource Modeling Rules

### 4.1 Agent Service

Use `resourceType: "agent_service"` for a callable Agent service.

The DID Document should include:

- semantic description in `oanMetadata.resourceDescription`;
- service endpoint for invocation or messaging;
- protocol binding such as HTTP, A2A, AIP, RPC, or another declared protocol;
- version information for package and protocol/interface where applicable;
- credential requirements when invocation needs authorization.

### 4.2 Skill

Use `resourceType: "skill"` for a portable capability description or Skill
artifact.

A Skill does not have to expose a network endpoint. If the Skill file or package
is hosted externally, put its URL and hash in `oanMetadata.packageInfo`.

Do not assume Discovery will host or proxy the Skill file. Discovery should
return the DID Document and verifiable artifact references.

### 4.3 MCP Server

Use `resourceType: "mcp_server"` for a server exposing MCP tools, resources, or
prompts.

The DID Document should include:

- MCP endpoint and transport in `service`;
- MCP protocol binding in `oanMetadata.protocolBindings`;
- semantic summary of exposed tools, resources, prompts, and safety boundaries.

### 4.4 Tool / API

Use `resourceType: "tool_api"` for an HTTP API, RPC interface, function service,
OpenAPI-described tool, or similar callable product.

The DID Document should include:

- API or RPC endpoint in `service`;
- schema or OpenAPI reference if available;
- semantic description of purpose, inputs, outputs, examples, and restrictions.

## 5. Minimal DID Document Shape

Generate DID Documents using this shape and adapt fields to the target resource.

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

For Agent Service, MCP Server, or Tool / API, change the DID subject code,
`resourceType`, service type, endpoint, and protocol binding accordingly.

## 6. Registration Workflow

Use the current resource-oriented API surface.

### Step 1: Prepare the DID Document

Create or obtain:

- `resourceDid`
- DID Document
- verification method and signing key
- `resourceType`
- package version
- package hash
- metadata hash
- subject-control proof or equivalent registration proof required by the
  Registrar

Validate before submission:

- DID starts with `did:oan:`.
- DID subject code matches `resourceType`.
- DID Document `id` equals `resourceDid`.
- `oanMetadata.resourceType` equals the submitted `resourceType`.
- external artifacts have hashes when they are referenced.
- version fields are explicit.

### Step 2: Register With Registrar

Submit the resource to the Registrar.

```http
POST /resources/register
Content-Type: application/json
```

The request body should include at least:

```json
{
  "resourceDid": "did:oan:SKFI:REPLACE_WITH_32_CHAR_SUFFIX",
  "resourceType": "skill",
  "didDocument": {},
  "metadata": {},
  "signature": "REPLACE_WITH_SIGNATURE"
}
```

If the Registrar uses a two-step flow, create or update the draft first, then
submit the completed resource.

```http
POST /resources/submit
Content-Type: application/json
```

### Step 3: Root Verification and Publication

The Registrar submits the resource to Root, or the operator triggers the Root
publish flow.

```http
POST /root/resources/verify-and-publish
Content-Type: application/json
```

Root verifies the resource, creates a Root proof, archives the version, and
queues downstream publication.

### Step 4: CDN Publish and Discovery Visibility

After Root verification, Root publishes a CDN event through its outbox and NATS
JetStream. `cdn-publisher` consumes the event, publishes the package to CDN,
and marks it published at Root. Root then sends authorized resource summaries to
Discovery through `/discovery/resources/sync-authorized`.

The resource is discoverable only after Root verification, CDN publication, and
Discovery indexing have completed. Clients and tests should observe status or
query results rather than manually driving Root batch endpoints.

## 7. Discovery Workflow

To discover resources, query Discovery with a user need, capability tags, and
optional resource type.

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

Discovery should return resource candidates with:

- `resourceDid`
- `resourceType`
- resource description
- lifecycle state
- version information
- package or artifact references
- Root proof material where available
- service or protocol bindings where available

## 8. Verification Workflow

Before recommending a resource for use, verify:

- the DID is a valid `did:oan` identifier;
- DID subject code matches `resourceType`;
- DID Document `id` equals `resourceDid`;
- Root proof binds `resourceDid`, `resourceType`, version, package hash, and
  metadata hash;
- downloaded artifacts match declared hash values;
- lifecycle state is active;
- credential requirements are understood before download or invocation;
- endpoint or manifest references come from the resolved DID Document or
  Root-approved package metadata.

Do not treat semantic description as proof of quality, safety, or correctness.
OAN verifies identity, publication, authorization, and integrity facts; it does
not prove that a model, Skill, server, or API is semantically honest.

## 9. Common Assistant Tasks

### 9.1 Create a Resource Registration Draft

When the user says "help me register this Agent/Skill/MCP/API in OAN":

1. Identify the resource type.
2. Collect missing metadata.
3. Generate the DID Document draft.
4. Generate the resource registration payload draft.
5. Point out placeholders requiring real keys, signatures, URLs, or hashes.
6. Submit only if the user has provided target endpoints and credentials.

### 9.2 Discover a Resource

When the user says "find a resource in OAN":

1. Convert the user need into a Discovery query.
2. Add `resourceType` if the user knows the product form.
3. Query Discovery.
4. Summarize candidates by name, description, type, version, endpoint or
   artifact reference, and verification state.
5. Recommend verification before use.

### 9.3 Verify a Candidate

When the user says "can I use this candidate":

1. Resolve or fetch the candidate DID Document.
2. Check DID and resource type consistency.
3. Check Root proof and package hash.
4. Check lifecycle state.
5. Check credential requirements.
6. Explain whether it is ready to download, invoke, or investigate further.

## 10. Do Not Do These Things

- Do not create new `did:ans` identifiers.
- Do not use `ansMetadata` in new DID Documents.
- Do not call old Agent-only public routes such as `/agents/draft` or
  `/root/agents/verify-and-publish`.
- Do not assume Discovery hosts Skill files or external artifacts.
- Do not invent private keys, signatures, hashes, credentials, or Root proofs.
- Do not present Root proof as a guarantee of business quality or model safety.
- Do not silently change an exact version request to the latest version.

## 11. Expected Follow-Up Integrations

This Skill can later be complemented by:

- an OAN MCP Server exposing registration, discovery, and verification tools;
- OpenAPI documents for Registrar, Root, CDN, and Discovery APIs;
- SDK helpers for `publishSkill`, `publishMcpServer`, `publishToolApi`, and
  `verifyResourceCandidate`;
- website flows that generate the same DID Document and registration payloads.

