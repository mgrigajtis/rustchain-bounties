# RustChain Troubleshooting (v2)

This doc covers common RustChain v2 issues for miners and users verifying balances.

---

## 1) "SSL certificate problem" / TLS verification failures

The public node may use a self-signed or otherwise-untrusted TLS cert.

Use `curl -k` (or `-sk`) to skip verification:

```bash
curl -sk https://50.28.86.131/api/status
```

---

## 2) Balance stays at 0.0 even though the miner is running

### First: confirm you’re checking the right miner ID

Your miner ID is the value you pass as `--wallet`.

Verify:

```bash
curl -sk "https://50.28.86.131/wallet/balance?miner_id=YOUR_MINER_ID" | jq
curl -sk "https://50.28.86.131/balance/YOUR_MINER_ID" | jq
```

### Second: check for VM/container fingerprint failure

If you run inside a VM/container, RustChain will often assign a near-zero weight.

Symptoms in miner output:
- `Fingerprint: FAILED`
- `VM/CONTAINER DETECTED - MINIMAL REWARDS`
- detection hints like `virtualbox` or `hypervisor`

Fix:
- Run the miner on **real hardware**.
- Avoid VPS/containers unless you know your provider exposes real hardware fingerprints.

---

## 3) Enroll errors

If enroll fails, you’ll see it right after attestation.

Things to try:
- Make sure you’re using the correct node URL: `https://50.28.86.131`
- Verify the node is healthy:

```bash
curl -sk https://50.28.86.131/health
curl -sk https://50.28.86.131/api/status | jq
curl -sk https://50.28.86.131/api/stats | jq
```

---

## 4) "Payout queued" but balance is still 0

Some bounties queue transfers for up to ~24 hours.

What to do:
1. Check your balance as above.
2. If a maintainer provided a **pending ID** (e.g. `Pending ID: 23`), wait for the confirmation window.
3. If balance is still 0 after the stated window, reply to the bounty issue with:
   - your wallet ID
   - the output of `wallet/balance`
   - the pending ID / tx hash

---

## 5) How to check node status

```bash
curl -sk https://50.28.86.131/api/status | jq
```

Example response:

```json
{
  "block_height": 10041,
  "chain_id": "rustchain-mainnet-v2",
  "epoch": 69,
  "sync_status": "synced",
  "ts": 1770731700,
  "version": "2.2.1-security-hardened"
}
```

---

## 6) Useful endpoints

- Status: `GET /api/status`
- Stats: `GET /api/stats`
- Epoch: `GET /epoch`
- Balance (wallet): `GET /wallet/balance?miner_id=...`
- Balance (miner): `GET /balance/{miner_pk}`
- Withdraw history: `GET /withdraw/history/{miner_pk}`
