# RustChain Miner Setup Guide (v2)

This guide walks you through setting up a RustChain v2 miner and verifying that it is actually earning RTC.

> TL;DR
> - **Mine on real hardware.** VMs/containers are detected and receive **near-zero rewards**.
> - Your "wallet" for mining is your **miner ID** (string) you pass to the miner.
> - Verify earnings with:
>   - `curl -sk "https://50.28.86.131/wallet/balance?miner_id=YOUR_MINER_ID"`

---

## 0) Requirements & common gotchas

### Real hardware required (important)

RustChain mining uses **Proof-of-Attestation** and hardware fingerprinting. If you run inside a VM/container (VirtualBox, VMware, Docker, many VPS providers), fingerprint checks may fail and you can receive minimal rewards.

If your miner prints any of these, you are effectively mining for free:
- `VM/CONTAINER DETECTED - MINIMAL REWARDS`
- `Fingerprint: FAILED`
- `cpuinfo:hypervisor`
- `product_name: virtualbox`

### Tools

- Python **3.10+** recommended
- `curl` (for verification)
- Optional: `jq` for nicer JSON output

---

## 1) Choose a Miner ID (wallet)

A Miner ID is an identifier string used by the network to track your balance.

Examples:
- `my-miner-01`
- `matthewRig`
- `agentgubbins`

> Tip: keep it consistent. If you change your miner ID, you are mining to a different "wallet".

---

## 2) Download the official miner

### Linux / macOS

```bash
mkdir -p ~/rustchain
cd ~/rustchain
curl -sk https://50.28.86.131/miner/download -o rustchain_miner.py
chmod +x rustchain_miner.py
```

### Windows (PowerShell)

```powershell
mkdir $HOME\rustchain
cd $HOME\rustchain
curl.exe -k https://50.28.86.131/miner/download -o rustchain_miner.py
```

---

## 3) Run the miner

### Linux / macOS

```bash
python3 ./rustchain_miner.py --wallet YOUR_MINER_ID --node https://50.28.86.131
```

### Windows

```powershell
py .\rustchain_miner.py --wallet YOUR_MINER_ID --node https://50.28.86.131
```

What you should see:
- Attestation challenge + submit
- Enrollment into current epoch
- A loop that runs on the block/epoch cadence

---

## 4) Verify your miner is enrolled & earning

### Check current epoch

```bash
curl -sk https://50.28.86.131/epoch | jq
```

### Check your balance

Two common balance endpoints exist; use whichever you prefer:

```bash
# "Wallet" balance endpoint (commonly referenced in bounties)
curl -sk "https://50.28.86.131/wallet/balance?miner_id=YOUR_MINER_ID" | jq

# Miner balance endpoint (OpenAPI)
curl -sk "https://50.28.86.131/balance/YOUR_MINER_ID" | jq
```

### Check withdrawal history

```bash
curl -sk "https://50.28.86.131/withdraw/history/YOUR_MINER_ID" | jq
```

---

## 5) Running the miner 24/7

### Linux systemd (recommended)

Create a service file:

```bash
sudo tee /etc/systemd/system/rustchain-miner.service >/dev/null <<'UNIT'
[Unit]
Description=RustChain v2 Miner
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=%i
WorkingDirectory=%h/rustchain
ExecStart=/usr/bin/python3 %h/rustchain/rustchain_miner.py --wallet YOUR_MINER_ID --node https://50.28.86.131
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
UNIT
```

Then enable + start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now rustchain-miner.service
sudo journalctl -u rustchain-miner.service -f
```

> If you don’t want system-wide services, you can also run via `screen`/`tmux`.

---

## 6) Troubleshooting

See `docs/troubleshooting.md` for common errors (TLS, fingerprint failures, enroll errors, and how to confirm you’re earning).
