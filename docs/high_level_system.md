# Dula Payments Platform — High-Level System Design

## 1. Vision & Scope
- Deliver a Kenyan-focused payments super-app that blends Mpesa interoperability with an on-chain USDC rail on Arbitrum.
- Support merchants and consumers end to end: onboarding, collections, payouts, P2P transfers, and cash in/out via agent network.
- Provide privacy-aware public identifiers (phone, email, username, or system ID) and optional DeFi yield for business balances via account abstraction wallets.

## 2. Core Personas & Journeys
1. **Consumer (P2P)**
   - Registers with phone/email, selects public handle, links Mpesa.
   - Sends/receives via phone, email, or username; chooses rail (Mpesa, on-chain, internal ledger).
2. **Merchant (SME, PoS)**
   - Onboards business profile, selects public identifier, configures payout accounts (Mpesa Till/PayBill, bank, on-chain).
   - Accepts payments through QR, payment links, PoS API, or pochi-like flows.
   - Off-ramps to Mpesa/business bank; allocates float to DeFi yield vaults.
3. **Field Agent**
   - Cash in/out desk; handles KES ↔ USDC conversions; settles with treasury wallet.

## 3. Payment Flows (Happy Path)
1. **Merchant Collection (Mpesa-lite/Pochi)**
   - Customer scans QR / enters merchant identifier.
   - Payment router decides rail: (a) push STK to Mpesa API, (b) internal ledger transfer if both users on Dula, (c) on-chain USDC transfer via Arbitrum smart contract.
   - Merchant receives confirmation + receipt showing chosen public handle.
2. **P2P Transfer**
   - Sender selects recipient identifier and funding source (Mpesa wallet, Dula balance, on-chain AA wallet).
   - If Mpesa: initiate STK push / B2C payout via Mpesa API.
   - If on-chain: abstracted smart account signs meta-tx; relayer submits on Arbitrum; state syncs back to ledger.
3. **Merchant Off-Ramp**
   - Merchant requests payout to Mpesa Till/PayBill or phone.
   - Treasury service batches requests, executes Mpesa B2B/B2C APIs, updates settlement ledger.
4. **Agent Cash-In**
   - User deposits KES; agent uses agent portal to log transaction; treasury issues USDC (custodial) to user wallet; backend reconciles cash float nightly.

## 4. System Architecture Overview
```
[Client Apps]
  - Web Dashboard (React) | Mobile (React Native)
      ↓ GraphQL/REST (HTTPS)
[API Gateway]
      ↓
[Backend Services (Node/Express)]
  - Identity & KYC
  - Ledger & Wallets
  - Payment Orchestrator
  - Treasury & Liquidity
  - Notification Service
      ↓
[Integrations]
  - Mpesa API (STK, B2C, B2B)
  - Arbitrum Smart Contracts (USDC vault, account abstraction bundler)
  - Analytics/BI, Compliance tools
[Storage]
  - MongoDB (core data)
  - Redis (sessions, rate limits)
  - Object storage (receipts, docs)
```

## 5. Component Details
### 5.1 Client Applications
- **Merchant Dashboard (React/Next.js)**: registration, payout rules, reporting, DeFi controls.
- **Consumer App (React Native)**: onboarding, P2P, agent locator, QR payments.
- **Agent Portal (PWA)**: float reporting, KYC capture, settlement tracking.

### 5.2 Backend (MERN)
- **API Gateway**: Express-based, handles auth (JWT + OAuth for partners), rate limiting, request signing.
- **Identity & KYC Service**: OTP verification, document upload, integration with KRA/credit bureaus, assigns public identifiers.
- **Ledger & Wallet Service**: double-entry ledger for KES tokens and USDC balances; tracks merchant sub-accounts and pooled treasury wallets.
- **Payment Orchestrator**: state machine driving Mpesa flows, on-chain relays, retries, and reconciliation; emits events to Kafka (optional) for audit.
- **Treasury & Liquidity**: manages float across Mpesa tills, bank accounts, Arbitrum smart contracts, and agent inventory.

### 5.3 Blockchain Layer (Arbitrum One)
- **Account Abstraction**: smart accounts per user via thirdweb/alchemy; support session keys for merchant PoS.
- **USDC Custodial Vault**: contracts holding pooled USDC; optional yield strategies (Aave, native yield) gated per merchant policy.
- **Settlement Contract**: logs transfers, emits events for backend ingestion.

### 5.4 Mpesa Integration
- Use Safaricom Daraja APIs for:
  - STK Push (C2B) for consumer funding.
  - B2C/B2B payouts for merchant settlements and agent float top-ups.
  - Reconciliation via Transaction Status API.
- Build webhook receiver for payment confirmations and register callback URLs per Till/PayBill.

### 5.5 Agent Network
- Each agent assigned float account (KES + USDC) and compliance tier.
- Support NFC/smartcard or OTP confirmation for physical cash exchanges.
- Daily settlement between agent Mpesa till and Dula treasury; alerts for float breaches.

## 6. Data Model Highlights
- **User**: PII, handles, KYC status, linked wallets, notification preferences.
- **Merchant**: business metadata, payout destinations, allowed rails, fee tiers.
- **Wallet**: type (consumer, merchant, agent, treasury), currency (KES, USDC), balance, AA address.
- **Transaction**: ledger entries, rail metadata (Mpesa receipt, tx hash), compliance flags.
- **Agent Float Log**: cash movements, reconciled status, supervisor approvals.

## 7. Security & Compliance
- Enforce tiered KYC (Tier 0: phone verify, Tier 1: ID+Selfie, Tier 2: business docs).
- Mask phone numbers in receipts unless explicitly shared; use hashed identifiers internally.
- Apply AML/CTF rules engine; integrate with sanctions/PEP screening.
- Encrypt sensitive data at rest (MongoDB field-level) and in transit (TLS 1.2+); secure secret storage via Vault/KMS.
- Implement transaction signing + 2FA for high-value merchant actions.

## 8. Observability & Ops
- Centralized logging (ELK/Datadog), structured events per payment stage.
- Metrics: success rates per rail, ledger-Mpesa reconciliation delta, agent float utilization, smart contract events per minute.
- Alerting for stuck payouts, on-chain relayer backlog, FX spreads deviating > threshold.

## 9. Roadmap (Phase 0 → Phase 2)
1. **Phase 0 – Foundations (0-2 mo)**
   - MERN skeleton, basic auth, sandbox Mpesa integration, custodial USDC ledger (no on-chain yet).
2. **Phase 1 – Dual Rail Beta (2-6 mo)**
   - Launch P2P + merchant pochi-like payments, introduce Arbitrum AA wallets, ship agent portal MVP.
3. **Phase 2 – Scale & DeFi (6-12 mo)**
   - Add DeFi yield options, auto-sweeps, Airtel/bank integrations, advanced analytics, automated treasury balancing.

## 10. Open Questions / Next Steps
- Finalize identifier policy (can merchants change handle post-registration?).
- Decide on custodial vs non-custodial balance for consumer AA wallets.
- Choose USDC yield venue and risk controls; evaluate launching a KES stablecoin later.
- Define agent compensation model and liquidity SLAs.
- Regulatory pathway: know required CBK/KPA licenses and sandbox participation.
