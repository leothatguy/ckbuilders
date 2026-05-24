# Builder Track Weekly Report - Week 10

**Name:** Divine Oshione Anesi

**Week Ending:** 05-23-2026

---

## Starting Point

At the start of the week, Loavix was working as a wallet-facing CKB dApp on local offckb devnet:

- The frontend could connect a wallet, create a draft invoice, build a CKB invoice-cell transaction, sign it from the browser, and attach the resulting outpoint to the backend.
- The backend could verify wallet-issued invoice cells and index chain-derived invoice state.
- The next Web3 work was to harden wallet identity, move the invoice script to CKB testnet, and integrate Fiber as an off-chain CKB payment path.

---

## Web3 Topics Applied

- Wallet-native authentication:
  - Challenge -> wallet signature -> backend verification -> short-lived session
  - CKB wallet/lock hash as the application identity
  - Merchant wallet authorization for invoice mutations
- CKB testnet script deployment:
  - Testnet deployer wallet setup
  - Testnet CKB funding checks
  - Invoice script deployment and metadata wiring
  - Frontend/backend testnet network alignment
- Fiber Network:
  - FNN receiver node setup
  - FNN payer node setup
  - Fiber invoice generation
  - Fiber channel opening
  - Fiber payment routing and settlement verification
- Fiber integration architecture:
  - Private FNN RPC behind the backend
  - Public browser flow exposing only the `fibt...` invoice address

---

## Resources Reviewed

This week I focused on the operational resources needed to run and integrate Fiber with a real CKB invoice flow:

- [nervosnetwork/fiber](https://github.com/nervosnetwork/fiber)
  - Confirmed FNN as the current reference implementation for Fiber.
  - Reviewed FNN key storage, RPC behavior, node startup, and CLI usage.
- [Fiber Public Nodes / Connect Nodes guide](https://www.fiber.world/docs/quick-start/connect-nodes)
  - Used the public-node flow to connect testnet FNN nodes.
  - Confirmed channel opening and `ChannelReady` verification.
- [Fiber Desktop discussion](https://talk.nervos.org/t/fiber-desktop-run-fiber-fnn-on-your-laptop-without-the-public-node-headache/10247)
  - Reviewed the current direction for non-technical users: a desktop app wrapping FNN.
- [Fiber Checkout completion report](https://talk.nervos.org/t/spark-program-fiber-checkout-a-stripe-style-react-payment-library-for-fiber-network/10045)
  - Reviewed the checkout pattern: browser UI -> server-side proxy/backend -> private FNN.
- [Fiber-checkout repository](https://github.com/salmansarwarr/Fiber-checkout)
  - Checked how a checkout component can generate invoices, display QR/payment data, and poll payment state without exposing the raw node RPC.

---

## Key Learnings

- Wallet-native identity maps cleanly to Loavix because invoice ownership is already expressed by the merchant's CKB lock/address.
- Challenge-based wallet auth still needs strict replay protection: signed challenges should be short-lived, single-use, and deleted after verification.
- For Loavix authorization, the critical rule is simple:

```txt
connected wallet identity must match invoice.merchantAddress
```

- Moving from devnet to testnet requires more than changing RPC URLs. The invoice script must be deployed to testnet and the exact script metadata must be used by the app.
- Fiber payments are not currently a normal MetaMask/JoyID-style browser payment. The payer needs FNN, Fiber Desktop, fiber-pay, or another Fiber-compatible payment client.
- The correct Loavix Fiber architecture is:

```txt
User browser -> Loavix frontend -> Loavix backend -> private merchant FNN
```

- FNN RPC should not be exposed publicly. Users should only receive the generated `fibt...` invoice address or QR/payment data.
- A single FNN cannot normally pay its own invoice. A real Fiber test needs another payer node or wallet with channel liquidity.

---

## Practical Progress

### Wallet Identity and Authorization

- Added a wallet-based challenge/sign/verify flow for Loavix.
- Used wallet signatures as proof of identity.
- Added session handling around verified wallet identity.
- Added authorization rules so invoice mutations can be tied to the merchant wallet.
- Fixed repeated wallet-sign prompts after frontend state refreshes.
- Confirmed wallet auth works without repeated signing prompts after the first successful verification.

### CKB Testnet Invoice Script

- Prepared and funded a testnet deployer wallet.
- Deployed the Loavix invoice script to CKB testnet.
- Added testnet script metadata to the app configuration.
- Confirmed the app is using the testnet invoice script metadata:

```txt
network: testnet
rpcUrl: https://testnet.ckb.dev
invoice script code hash: 0x07ec5974d5c4297494de30b9be23521d4df05dc4d31879418f985e5da865aa14
```

### Fiber Receiver Node

- Set up a local merchant/receiver FNN node.
- Funded the receiver node with testnet CKB.
- Connected the receiver node to Fiber testnet peers.
- Opened a Fiber channel to a public testnet peer.
- Confirmed the receiver channel reached:

```txt
ChannelReady
```

### Loavix Fiber Integration

- Added backend support for creating Fiber invoices through FNN.
- Added backend support for checking Fiber invoice status.
- Stored Fiber payment references on Loavix invoices:
  - `fiberPaymentHash`
  - `fiberInvoiceAddress`
  - `fiberPreimage`
- Added a payer-page flow to:
  - prepare a Fiber payment
  - display the generated `fibt...` invoice address
  - check Fiber payment status
- Confirmed Loavix can generate a real Fiber invoice address through the backend.

### Fiber Payer Node and Payment Test

- Set up a second local FNN node as the payer.
- Used separate local ports for the two nodes:

```txt
merchant FNN RPC: 127.0.0.1:8227
payer FNN RPC:    127.0.0.1:8229
merchant P2P:     8228
payer P2P:        8230
```

- Funded the payer node with testnet CKB.
- Opened a public channel from the payer node to a Fiber peer.
- Connected the payer node directly to the merchant node for a controlled local test.
- Opened a direct payer-to-merchant private Fiber channel.
- Confirmed the direct channel reached:

```txt
ChannelReady
```

### End-to-End Fiber Payment Proof

- Generated a Loavix Fiber invoice:

```txt
fibt...
```

- Paid the invoice from the payer FNN node.
- Confirmed payer-side payment status:

```txt
Success
```

- Confirmed merchant-side Fiber invoice status:

```txt
Paid
```

- Confirmed Loavix invoice status:

```txt
PAID
```

- Verified the paid Loavix invoice id:

```txt
d87fe8c7-333f-4f58-b9ef-fa08236b5295
```

- Verified the Fiber payment hash:

```txt
0xb04bb2546f3ad17238efdd98bb08845c3fb7bfa230f1cac5d98073131ab1c9a4
```

This proved the full path:

```txt
Loavix invoice -> FNN invoice -> payer FNN payment -> merchant FNN Paid -> Loavix PAID
```

## Verification

Key Web3/Fiber checks completed:

```bash
docker exec loavix-fiber fnn-cli info
docker exec loavix-fiber fnn-cli channel list_channels
docker exec loavix-fiber-payer fnn-cli -u http://127.0.0.1:8229 channel list_channels
docker exec loavix-fiber-payer fnn-cli -u http://127.0.0.1:8229 payment send_payment --invoice '<fibt...>'
docker exec loavix-fiber fnn-cli -u http://127.0.0.1:8227 invoice get_invoice --payment-hash <hash>
```

Confirmed states:

```txt
Fiber channel: ChannelReady
Fiber invoice: Paid
Loavix invoice: PAID
```

---

## Next Deployment Step

The Fiber update was not hosted this week. The work completed here was a local testnet proof using local FNN nodes and CKB testnet. The planned hosted testnet shape for the next step is:

```txt
Browser -> Vercel frontend -> backend API -> private FNN RPC
```

For Fiber, the backend and FNN should run on the same machine or private network:

- Backend is public.
- FNN P2P port is reachable by the Fiber network.
- FNN RPC is private to the backend.
- Users only see the generated `fibt...` invoice address.

The payer-side limitation remains: users need FNN, Fiber Desktop, fiber-pay, or another Fiber-compatible client to pay `fibt...` invoices until browser wallet support matures.

---

## Environment

- CKB testnet RPC: `https://testnet.ckb.dev`
- Merchant FNN RPC: `http://127.0.0.1:8227`
- Payer FNN RPC: `http://127.0.0.1:8229`
- Fiber Docker image: `nervos/fiber:0.9.0-rc1`
- JavaScript SDK: CCC (≥1.12.5)
- Backend framework: NestJS
- Frontend framework: Next.js
