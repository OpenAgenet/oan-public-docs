<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

# did-oan

The current OAN implementation target is
[`OAN DID Method Specification.md`](doc/OAN%20DID%20Method%20Specification.md).

The specification uses the profile with a five-character `routing-code` and a
32-character `suffix-code`. The former four-character `semantic-code` syntax is
retained only as migration evidence or negative-test input; new SDKs,
Registrars, Root services, fixtures, and production deployments MUST NOT
generate or accept it as the active profile.
