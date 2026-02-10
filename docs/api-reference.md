# RustChain API Reference (v2)

This document describes the **public HTTP API** for the RustChain v2 Integrated Server.

Source of truth: the server’s OpenAPI spec at:

- `https://50.28.86.131/openapi.json`

> Notes
> - Most endpoints are unauthenticated.
> - The API is HTTPS with a self-signed/unknown CA in many environments. The examples below use `curl -k` to skip TLS verification.

---

## Base URL

```
https://50.28.86.131
```

## Health / status

The following are useful operational endpoints. Some may not be listed in OpenAPI.

### GET `/health`

```bash
curl -sk https://50.28.86.131/health
```

### GET `/api/status`

Returns current sync / height / version.

```bash
curl -sk https://50.28.86.131/api/status | jq
```

---

## Stats

### GET `/api/stats`

Returns basic chain/network stats.

```bash
curl -sk https://50.28.86.131/api/stats | jq
```

---

## Epochs

### GET `/epoch`

Returns the current epoch info.

```bash
curl -sk https://50.28.86.131/epoch | jq
```

### POST `/epoch/enroll`

Enroll a miner/device into epoch rewards.

```bash
curl -sk -X POST https://50.28.86.131/epoch/enroll \
  -H 'Content-Type: application/json' \
  -d '{
    "miner_pk": "YOUR_MINER_ID_OR_PUBKEY",
    "device_fingerprint": "YOUR_DEVICE_FINGERPRINT"
  }' | jq
```

> If you don’t know your `device_fingerprint`, use the official miner to generate it.

---

## Balances

Two balance endpoints are commonly referenced.

### GET `/balance/{miner_pk}`

Returns the balance for a miner identifier (listed in OpenAPI).

```bash
curl -sk "https://50.28.86.131/balance/YOUR_MINER_ID" | jq
```

### GET `/wallet/balance?miner_id=...`

Returns the balance for a miner identifier (frequently used in bounty verification comments).

```bash
curl -sk "https://50.28.86.131/wallet/balance?miner_id=YOUR_MINER_ID" | jq
```

---

## Attestation

### POST `/attest/challenge`

Requests an attestation challenge.

```bash
curl -sk -X POST https://50.28.86.131/attest/challenge \
  -H 'Content-Type: application/json' \
  -d '{
    "miner_pk": "YOUR_MINER_ID_OR_PUBKEY"
  }' | jq
```

### POST `/attest/submit`

Submits an attestation response.

```bash
curl -sk -X POST https://50.28.86.131/attest/submit \
  -H 'Content-Type: application/json' \
  -d '{
    "miner_pk": "YOUR_MINER_ID_OR_PUBKEY",
    "challenge": "CHALLENGE_FROM_PREVIOUS_STEP",
    "response": "YOUR_ATTESTATION_RESPONSE"
  }' | jq
```

---

## Withdrawals

Withdrawals are used to move funds out of a miner balance via a registered address.

### POST `/withdraw/register`

Registers a withdrawal destination.

```bash
curl -sk -X POST https://50.28.86.131/withdraw/register \
  -H 'Content-Type: application/json' \
  -d '{
    "miner_pk": "YOUR_MINER_ID_OR_PUBKEY",
    "destination": "YOUR_DESTINATION_ADDRESS"
  }' | jq
```

### POST `/withdraw/request`

Requests a withdrawal.

```bash
curl -sk -X POST https://50.28.86.131/withdraw/request \
  -H 'Content-Type: application/json' \
  -d '{
    "miner_pk": "YOUR_MINER_ID_OR_PUBKEY",
    "amount": 1.0
  }' | jq
```

### GET `/withdraw/status/{withdrawal_id}`

Checks the status of a withdrawal.

```bash
curl -sk "https://50.28.86.131/withdraw/status/WITHDRAWAL_ID" | jq
```

### GET `/withdraw/history/{miner_pk}`

Lists withdrawal history for a miner.

```bash
curl -sk "https://50.28.86.131/withdraw/history/YOUR_MINER_ID" | jq
```

---

## Metrics

### GET `/metrics`

Prometheus-style metrics.

```bash
curl -sk https://50.28.86.131/metrics | head
```

---

## Appendix: OpenAPI extraction

If you want to regenerate this doc from the OpenAPI file:

```bash
curl -sk https://50.28.86.131/openapi.json -o openapi.json
# then inspect paths:
python3 - <<'PY'
import json
j=json.load(open('openapi.json'))
print('\n'.join(sorted(j['paths'].keys())))
PY
```
