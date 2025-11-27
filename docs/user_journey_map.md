# Dula User Journey Map

## 1. Consumer Journey (P2P & Everyday Payments)
### Phase A: Awareness & Onboarding
1. Discovers Dula via referral/social media.
2. Installs app or opens web onboarding page.
3. Provides phone/email, receives OTP, chooses preferred public identifier (phone, email, username, or system ID).
4. Completes Tier-0 KYC (basic info); optional Tier-1 (ID + selfie) for higher limits.
5. Links Mpesa wallet and optional bank card; guided tour of dashboard.

### Phase B: Activation
1. Adds funds by triggering Mpesa STK push or converts existing on-chain USDC via QR.
2. Sends first P2P transfer to contact handle; sees instant receipt with masked identifiers.
3. Sets up notifications (push/SMS/email) for incoming payments.

### Phase C: Habitual Usage
1. Uses QR scanner at merchants; selects rail (Mpesa vs Dula balance vs USDC).
2. Saves favorite merchants; receives loyalty/reward prompts.
3. For cash needs, locates nearby agent via map, generates OTP for cash-out.

### Phase D: Retention & Growth
1. Receives monthly statement summarizing spending across rails.
2. Upsells to Tier-2 KYC for higher limits, DeFi access, multi-currency wallets.
3. Invites friends via referral code; tracks bonuses.

## 2. Merchant Journey (SME / PoS)
### Phase A: Onboarding
1. Signs up on merchant dashboard; provides business details and KRA PIN.
2. Chooses public display identifier (phone/email/custom handle/system ID).
3. Uploads compliance documents; awaits verification (Tier-2 KYC).
4. Configures payout destinations: Mpesa Till/PayBill, bank account, Arbitrum wallet.

### Phase B: Setup & Integration
1. Generates QR codes/payment links; embeds API keys into PoS/web store.
2. Defines settlement preferences (auto vs manual, frequency, currency mix).
3. Enables DeFi vault access (optional) and sets allocation percentage of idle balance.

### Phase C: Live Operations
1. Accepts payments via pochi-like flows, QR, PoS API, or on-chain invoices.
2. Dashboard shows real-time ledger, pending settlements, and yield earnings.
3. Initiates off-ramps to Mpesa or bank; monitors notifications for reconciliation.

### Phase D: Expansion & Support
1. Adds staff/roles with scoped permissions (cashier, accountant).
2. Requests working capital advance backed by transaction history.
3. Accesses analytics to optimize pricing; integrates with ERP/accounting systems.

## 3. Agent Journey (Cash In/Out Network)
### Phase A: Recruitment & Training
1. Applies via agent portal; submits KYC + business license.
2. Receives training on compliance, float management, and Dula tools.
3. Assigned agent ID, float limits, and Mpesa till linkage.

### Phase B: Daily Operations
1. Logs into portal each morning, verifies float (KES + USDC) balances.
2. Serves walk-in customers for deposits/withdrawals; uses OTP or NFC confirmation.
3. Records cash movements; system updates treasury ledger and triggers Mpesa B2B if float top-up needed.

### Phase C: Settlement & Compliance
1. At day end, reviews transaction log; confirms totals with Dula treasury team.
2. Handles discrepancy flags; uploads receipts if requested.
3. Receives commission payouts weekly via Mpesa or Dula wallet.

## 4. Internal Treasury & Support Journey
1. Treasury analysts monitor inflows/outflows across Mpesa, banks, Arbitrum vaults.
2. Automated rules rebalance float; manual overrides triggered via dashboard.
3. Support team tracks tickets, disputes, and compliance escalations tied to user journeys above.

## 5. Cross-Journey Touchpoints
- **Notifications**: Push/SMS/email for onboarding milestones, payments, settlements.
- **Compliance Checks**: Real-time screening at onboarding; ongoing monitoring for risky behaviors.
- **Analytics**: Journey funnels feed BI dashboards to track drop-off (e.g., Tier-1 completion rate, merchant activation time).
- **Feedback Loops**: In-app surveys after critical events (first payment, settlement) to refine UX.

