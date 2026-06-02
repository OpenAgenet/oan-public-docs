<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

# OAN Architecture Overview

OpenAgenet (Open Agent-Internet Network, OAN; Chinese name: "开原智网") is an
Agent Internet reference project built around `did:ans`, Root-governed
registration, verifiable discovery, trusted distribution, and signed
Agent-to-Agent invocation.

This public page gives the high-level module map and operating model. Detailed
design records are maintained outside the organization implementation
repositories.

## Repository Layers

OAN is split into focused repositories:

- `oan-protocol-common`: reusable Rust protocol crates, schemas, protocol
  constants, DID helpers, credential helpers, signing helpers, package models,
  and storage helpers.
- `oan-reference-services`: Rust reference services for Root, Registrar,
  Discovery, and CDN roles.
- `oan-agent-py`: Python reference Service Agent and User Agent implementations.
- `oan-sdk-ts`: TypeScript SDK packages for browser, Node.js, developer tools,
  and web-console usage.
- `oan-examples`: runnable registration, discovery, trusted invocation, and
  negative security examples.
- `oan-deploy`: container and deployment assets.
- `oan-site`: public website and high-level project documentation.
- `oan-trial-network`: public trial network information and community node
  participation workflow.
- `oan-operator-guides`: operator runbooks, checklists, and procedures.

## Runtime Roles

OAN currently models six runtime roles:

- Root Node: governance and trust anchor.
- Registrar Node: Service Agent onboarding and registration gateway.
- Discovery Node: verified Agent index and signed discovery response provider.
- CDN Service: package, DID Document, metadata, and manifest distribution layer.
- Service Agent: business Agent exposing callable capabilities.
- User Agent: business Agent or client that discovers and invokes Service
  Agents.

Root, Registrar, and Discovery are infrastructure roles. Service Agent and User
Agent are business Agent roles. CDN is a distribution service and is not trusted
as a protocol authority.

## Trust Flow

The reference flow is:

1. Root authorizes Registrar and Discovery participants.
2. Registrar helps a Service Agent prepare a complete DID Document and
   registration credential.
3. Root verifies the submitted package and appends governance facts to the
   bulletin.
4. Root publishes verified packages to CDN.
5. Root notifies authorized Discovery Nodes.
6. Discovery syncs packages from CDN and verifies Root proofs, hashes, and
   bulletin facts.
7. User Agent queries Discovery and receives a signed discovery response.
8. User Agent calls Service Agent with a signed trusted invocation envelope.
9. Service Agent verifies the caller DID Document, credential, nonce, timestamp,
   target DID, body hash, and request proof.
10. Service Agent returns a signed trusted invocation response.

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
- End-to-end and negative trusted invocation examples in `oan-examples`.
