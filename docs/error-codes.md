<!-- Copyright (c) 2026 OpenAgenet contributors -->
<!--
Initial author: JINLIANG XU
Email: jlxufly@gmail.com
-->

# Error Codes

This document records the current OpenAgenet security and protocol error code
vocabulary. APIs should return stable machine-readable codes whenever possible.

## General API shape

Infrastructure APIs currently return:

```json
{
  "error": "machine_readable_error_code"
}
```

Trusted invocation rejection currently returns:

```json
{
  "error": "trusted_invocation_rejected",
  "reason": "machine_readable_reason_code"
}
```

Future APIs should prefer:

```json
{
  "error": {
    "code": "machine_readable_error_code",
    "message": "human readable detail",
    "correlationId": "optional request id"
  }
}
```

## Trusted invocation reasons

- `invalid_invocation_type`
- `missing_invocation_fields`
- `target_did_mismatch`
- `missing_or_invalid_body_hash`
- `body_hash_mismatch`
- `invalid_timestamp`
- `timestamp_must_include_timezone`
- `timestamp_in_future`
- `timestamp_expired`
- `caller_did_document_mismatch`
- `credentials_must_be_array`
- `replayed_nonce`
- `request_signature_invalid`
- `missing_user_agent_credential`
- `user_credential_signature_invalid`

## Root verification errors

- `invalid_did`
- `invalid_did_document_structure`
- `invalid_subject_type`
- `invalid_service_endpoint`
- `invalid_registration_credential`
- `invalid_registration_credential_signature`
- `invalid_issuer_key`
- `invalid_request_signature`
- `invalid_nonce_store`

## Registrar draft validation errors

- `did_document_id_mismatch`
- `missing_did_document`
- `missing_registration_credential`

## Discovery package rejection reasons

- `package_decode_failed`
- `not_indexable`
- `invalid_did_document_hash`
- `invalid_metadata_hash`
- `invalid_root_proof`
- `unauthorized_domains`
- `invalid_root_key`

## Boundary rules

- Root verification errors describe governance and registration failures.
- Discovery rejection reasons describe local indexing and verification failures.
- Trusted invocation reasons describe peer-to-peer Agent access failures.
- CDN should not mint trust errors; relying parties must verify Root proof, hashes, and bulletin facts.


