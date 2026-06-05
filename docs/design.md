<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

# OAN Architecture Overview

OpenAgenet (Open Agent-Internet Network, OAN) is an Agent Internet reference
project built around `did:oan`, Root-governed registration, verifiable
discovery, trusted distribution, and signed cross-Agent or client-to-resource
invocation.

This public page gives the high-level module map and operating model. Detailed
design records are maintained outside the organization implementation
repositories.

## Repository Layers

OAN is split into focused repositories:

- `oan-protocol-common`: reusable Rust protocol crates, schemas, protocol
  constants, `did:oan` helpers, credential helpers, signing helpers, resource
  package models, and storage helpers.
- `oan-reference-services`: Rust reference services for Root, Registrar,
  Discovery, and CDN roles.
- `oan-agent-py`: Python reference Service Agent and User Agent implementations.
- `oan-sdk-ts`: TypeScript SDK packages for browser, Node.js, developer tools,
  and web-console usage.
- `oan-examples`: runnable resource registration, discovery, trusted
  invocation, and negative security examples.
- `oan-deploy`: container and deployment assets.
- `oan-site`: public website and high-level project documentation.
- `oan-trial-network`: public trial network information and community node
  participation workflow.
- `oan-operator-guides`: operator runbooks, checklists, and procedures.

## Runtime Roles

OAN currently models infrastructure roles and discoverable resource products:

- Root Node: governance and trust anchor.
- Registrar Node: onboarding and registration gateway for discoverable
  resources.
- Discovery Node: verified resource index and signed discovery response
  provider.
- CDN Service: package, DID Document, metadata, and manifest distribution layer.
- Resource Provider: the subject or operator that controls a registered
  resource.
- Resource Consumer: an Agent, application, developer tool, or human-facing
  client that discovers and uses resources.

Root, Registrar, Discovery, and future third-party VC issuers are authorized
infrastructure roles. CDN is a distribution service and is not trusted as a
protocol authority.

OAN's discoverable resource products include Agent Service, Skill, MCP Server,
and Tool/API. Each product is represented by a `did:oan` DID Document with
semantic description fields for discovery. Product-native artifacts, such as
Skill files or API descriptions, may be referenced by URI when they are not part
of the DID Document itself.

## Governance and Authorization

OAN separates governance state from protocol credentials:

- The on-chain governance layer records lifecycle state for authorized
  infrastructure participants, including Registrar, Discovery, and future
  third-party VC issuer nodes.
- The on-chain governance layer is not itself an authorization credential. It
  does not store resource DID Documents, resource packages, or VC bodies.
- Root observes the latest governance state through a local read-side view and
  issues, refuses, suspends, or revokes infrastructure authorization VCs.
- Effective infrastructure authorization requires both an active governance
  state and a valid Root-issued authorization VC. A VC alone is not sufficient
  after the governance state becomes inactive, and an active governance state
  alone is not enough before Root has issued the VC.
- Ordinary resource providers and consumers do not need to follow governance
  state directly by default. They rely on Root-signed packages, Registrar
  credentials, Discovery responses, and standard VC verification.

CDN is outside this authorization path. It only serves Root-published material,
and relying parties verify Root signatures and package hashes instead of
trusting CDN identity.

## Trust Flow

The reference flow is:

1. The governance process marks eligible Registrar, Discovery, or future
   third-party VC issuer participants as active in the on-chain governance
   layer.
2. Root observes the active governance state and issues Root authorization VCs
   after verifying node identity and DID-control materials.
3. Authorized Registrar and Discovery participants hold Root-issued
   infrastructure authorization VCs whose validity is also bounded by the latest
   governance state.
4. Registrar helps a resource provider prepare a complete `did:oan` DID
   Document and registration credential for an Agent Service, Skill, MCP Server,
   or Tool/API.
5. Root verifies the submitted package and records signed bulletin facts.
6. Root publishes verified packages to CDN.
7. Root notifies authorized Discovery Nodes.
8. Discovery syncs packages from CDN and verifies Root proofs, hashes, and
   bulletin facts.
9. Resource Consumer queries Discovery and receives a signed discovery response.
10. Resource Consumer uses the returned DID Document and referenced protocol
    material to call or obtain the resource.
11. Callable resources verify caller identity, credential material, nonce,
    timestamp, target DID, body hash, and request proof according to the
    resource protocol.

CDN is only a transport and storage layer. Relying parties verify Root-signed
material rather than trusting CDN directly.

## Public Usage Paths

For local use:

- Run infrastructure services from `oan-reference-services`.
- Run Python demo Agents from `oan-agent-py`.
- Run end-to-end examples from `oan-examples`.
- Use public protocol material from `oan-site/docs` and reusable code from
  `oan-protocol-common`.

For trial-network participation:

- Read node participation workflow in `oan-trial-network`.
- Use deployment assets from `oan-deploy`.
- Follow operational checklists in `oan-operator-guides`.

## Current Validation

The split repositories keep the existing local validation paths:

- Rust protocol crate tests in `oan-protocol-common`.
- Rust reference service tests in `oan-reference-services`.
- Python Agent compile and dependency checks in `oan-agent-py`.
- End-to-end, resource registration, discovery, and negative trusted invocation
  examples in `oan-examples`.
