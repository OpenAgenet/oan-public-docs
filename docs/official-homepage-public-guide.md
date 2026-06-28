<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

# Official OAN Homepage Public Guide

The official OAN homepage is the public entry point for explaining, registering,
discovering, and observing the OpenAgenet network.

Planned public URL:

```text
https://openagenet.xyz
```

## Top-Level Pages

The homepage uses five top-level pages:

- Home: project introduction, official positioning, and quick actions.
- Register: public resource registration entry through the official Registrar.
- Discover: public resource discovery entry through the official Discovery
  node.
- Docs: public documentation and specification gateway.
- Ecosystem: official-node status, ecosystem signals, and participation
  guidance.

Node status and participation guidance are part of Ecosystem rather than
separate top-level pages.

## Public Endpoint Model

The homepage uses a subdomain model:

| Role | Public URL |
|---|---|
| Homepage | `https://openagenet.xyz` |
| Homepage API | `https://api.openagenet.xyz` |
| Root | `https://root.openagenet.xyz` |
| Registrar | `https://registrar.openagenet.xyz` |
| Discovery | `https://discovery.openagenet.xyz` |
| Trust indexer | `https://trust.openagenet.xyz` |
| CDN | `https://cdn.openagenet.xyz` |

CDN is an OAN infrastructure component used by Root, publisher, Discovery,
operators, and diagnostics. Ordinary service-agent owners and user-agent users
should not need to understand or operate CDN directly.

## Registration Boundary

The Register page is intended to be public and official. A successful
registration submission means the resource has been accepted into the OAN
registration flow. It does not guarantee immediate Discovery visibility.

The user-visible lifecycle should distinguish:

- submitted to Registrar
- accepted by Root
- published/distributed by OAN
- visible in Discovery

Private keys and local identity backups must remain local. They should not be
uploaded to the homepage backend, Registrar, Discovery, Root, or CDN.

## Discovery Boundary

The Discover page queries an official Discovery node and presents verified
resource metadata and trust information. Discovery does not store private
service-agent credentials, and users should still verify signed resource
material according to OAN trust rules.

## Documentation Boundary

Public documentation should link only intentional public material, such as:

- `OpenAgenet/oan-public-docs`
- `OpenAgenet/oan-sdk-ts`
- public specifications, papers, and drafts intended for release

Internal research repositories, unpublished benchmarks, local demo artifacts,
private genesis material, and private design repositories should not be linked
from the public homepage.
