# Account Abstraction & Wallet Issuance

## Overview

NexusPay uses **ThirdWeb Smart Wallets** as the primary account abstraction framework for both user wallets and platform wallets. The system implements ERC-4337 account abstraction to provide gasless transactions, unified wallet addresses across chains, and enhanced security features.

## Account Abstraction Framework

### Primary Framework: ThirdWeb Smart Wallets

**Framework:** [ThirdWeb v5](https://thirdweb.com/) - Smart Wallet SDK  
**Standard:** ERC-4337 Account Abstraction  
**Package:** `thirdweb@^5.20.0` and `@thirdweb-dev/sdk@^4.0.99`

**Key Features:**
- Gasless transactions (gas sponsorship)
- Unified wallet addresses across all chains
- Smart contract wallets (not EOA)
- Multi-chain support (18+ chains)
- Default factory address (or custom factory)

### Secondary Framework: Biconomy (Legacy/Unused)

**Framework:** Biconomy Account Abstraction SDK  
**Package:** `@biconomy/account@^3.1.1-alpha.0`  
**Status:** Installed but **not actively used** in the codebase

Biconomy packages are present in `package.json` but the code has been migrated to ThirdWeb. The Biconomy imports are commented out or removed.

## Wallet Types

### 1. User Wallets (Personal Accounts)

**Type:** Smart Contract Wallets (ERC-4337)  
**Control:** Single EOA (Externally Owned Account)  
**Gas:** Sponsored (gasless for users)  
**Multi-chain:** Unified address across all chains

#### Wallet Creation Process

**Location:** `backend/src/services/auth.ts` - `createAccount()`

```typescript
export async function createAccount() {
    // Step 1: Create a random EOA (personal account)
    const newWallet = Wallet.createRandom();
    const pk = newWallet.privateKey;
    const personalAccount = privateKeyToAccount({
        client,
        privateKey: pk as string,
    });

    // Step 2: Define default chain (Ethereum mainnet)
    const defaultChain = defineChain(1); // Ethereum mainnet

    // Step 3: Configure smart wallet
    const wallet = smartWallet({
        chain: defaultChain,
        sponsorGas: true, // Enable gas sponsorship
        // Uses ThirdWeb's default factory if not specified
    });

    // Step 4: Connect smart wallet with personal account
    const smartAccount = await wallet.connect({
        client,
        personalAccount,
    });
    
    const walletAddress = smartAccount.address;

    return { pk, walletAddress };
}
```

#### When Wallets Are Created

**User Registration Flow** (`backend/src/controllers/authController.ts`):

1. User initiates registration → OTP sent
2. User verifies OTP
3. **Wallet created** via `createAccount()`
4. Wallet address stored in user record
5. Private key encrypted and stored (for recovery)

```typescript
// After OTP verification
const userSmartAccount = await createAccount();
const { pk, walletAddress } = userSmartAccount;

const newUser = new User({
    phoneNumber,
    email,
    walletAddress,  // Smart wallet address
    privateKey: pk, // EOA private key (for controlling smart wallet)
    isUnified: true // Mark as unified wallet
});
```

#### Key Characteristics

- **Unified Address:** Same address on all chains (Arbitrum, Polygon, Celo, etc.)
- **Gasless:** Users don't pay gas fees (sponsored by platform)
- **Smart Contract:** Wallet is a smart contract, not an EOA
- **Single Controller:** One EOA private key controls the smart wallet
- **Cross-chain:** Works seamlessly across 18+ blockchain networks

### 2. Platform Wallets (Multi-sig)

**Type:** Smart Contract Wallets with Multi-signature  
**Control:** 2-of-3 EOA accounts (multisig)  
**Gas:** Can be gasless (configurable)  
**Multi-chain:** Separate addresses per chain (or unified)

#### Wallet Creation Process

**Location:** `backend/src/services/platformWallet.ts` - `initializePlatformWallet()`

```typescript
export async function initializePlatformWallet(
  ownerKeys: string[], // Array of 3 private keys
  chainName: string = 'arbitrum'
): Promise<{ address: string }> {
    // Step 1: Create EOA accounts from private keys
    const ownerAccounts = ownerKeys.map(key => 
        privateKeyToAccount({ client, privateKey: key })
    );

    // Step 2: Define chain
    const chain = defineChain(chainConfig.chainId);

    // Step 3: Initialize smart wallet with custom factory
    const wallet = smartWallet({
        chain,
        factoryAddress: config.SMART_WALLET_FACTORY_ADDRESS || '',
        gasless: true, // Enable gasless transactions
    });

    // Step 4: Connect with first owner as primary
    const smartAccount = await wallet.connect({
        client,
        personalAccount: ownerAccounts[0],
    });

    return { address: smartAccount.address };
}
```

#### Multi-signature Setup

**Configuration:**
- **3 Owner Keys:** Primary, Secondary, Backup
- **Threshold:** 2-of-3 signatures required
- **Security:** Enhanced security for platform operations

**Environment Variables:**
```env
PLATFORM_WALLET_PRIMARY_KEY=primary_private_key
PLATFORM_WALLET_SECONDARY_KEY=secondary_private_key
PLATFORM_WALLET_BACKUP_KEY=backup_private_key
SMART_WALLET_FACTORY_ADDRESS=0x... # Custom factory (optional)
```

#### Custom Multisig Contract

**Location:** `backend/src/contracts/MultisigPlatformWallet.sol`

A custom Solidity contract for platform wallet multi-signature operations:
- 2-of-3 signature requirement
- Transaction queuing and confirmation
- Recovery mechanisms
- Reentrancy protection

**Note:** This contract exists but the platform primarily uses ThirdWeb's smart wallet implementation.

## Smart Wallet Factory

### Default Factory

**ThirdWeb Default Factory:**
- Automatically used if `SMART_WALLET_FACTORY_ADDRESS` is not set
- ThirdWeb manages the factory deployment
- Works across all supported chains

### Custom Factory

**Custom Factory Address:**
- Set via `SMART_WALLET_FACTORY_ADDRESS` environment variable
- Used for platform wallets
- Allows custom smart wallet implementations

**Configuration:**
```typescript
const wallet = smartWallet({
    chain,
    factoryAddress: config.SMART_WALLET_FACTORY_ADDRESS || '', // Custom or default
    gasless: true,
});
```

## Gas Sponsorship

### User Wallets

**Gasless Transactions:**
- Enabled by default: `sponsorGas: true`
- Users don't pay gas fees
- Platform sponsors all user transactions
- Works across all chains

**Configuration:**
```typescript
const wallet = smartWallet({
    chain: defaultChain,
    sponsorGas: true, // Gasless for users
});
```

### Platform Wallets

**Configurable Gas Sponsorship:**
- Can be enabled: `gasless: true`
- Or disabled for direct gas payment
- Controlled via environment variable: `ENABLE_GASLESS=true`

## Wallet Address Generation

### User Wallets

**Deterministic Address:**
- Generated from EOA address + factory address
- Same address across all chains (unified)
- Predictable and deterministic

**Process:**
1. Generate random EOA (personal account)
2. Connect to smart wallet factory
3. Factory deploys smart wallet contract
4. Smart wallet address = deterministic from EOA + factory

### Platform Wallets

**Per-Chain or Unified:**
- Can be unified (same address) or per-chain
- Depends on factory configuration
- Multi-chain initialization supported

## Integration Points

### 1. User Registration

**File:** `backend/src/controllers/authController.ts`

```typescript
// After OTP verification
const userSmartAccount = await createAccount();
const { pk, walletAddress } = userSmartAccount;
```

### 2. Token Transfers

**File:** `backend/src/services/token.ts`

```typescript
// Create personal account from private key
const personalAccount = privateKeyToAccount({
    client,
    privateKey: sourcePrivateKey
});

// Connect smart wallet
const wallet = smartWallet({
    chain,
    sponsorGas: true,
});

const smartAccount = await wallet.connect({
    client,
    personalAccount,
});

// Execute transfer
const transaction = transfer({
    contract,
    to: destinationAddress,
    amount,
});

await sendTransaction({
    transaction,
    account: smartAccount, // Smart wallet account
});
```

### 3. Platform Operations

**File:** `backend/src/services/platformWallet.ts`

```typescript
// Connect to platform smart wallet
const wallet = smartWallet({
    chain,
    factoryAddress: config.SMART_WALLET_FACTORY_ADDRESS || '',
    gasless: true,
});

const smartAccount = await wallet.connect({
    client,
    personalAccount: ownerAccounts[0], // Primary owner
});
```

### 4. M-Pesa Integration

**File:** `backend/src/controllers/mpesaController.ts`

Uses smart wallets for crypto releases:
- User receives crypto to their smart wallet
- Platform wallet uses smart wallet for transfers
- Gasless transactions for better UX

## Supported Chains

All wallet operations support **18+ blockchain networks**:

- Arbitrum (42161)
- Polygon (137)
- Base (8453)
- Optimism (10)
- Celo (42220)
- Scroll (534352)
- Fuse (122)
- Gnosis (100)
- Aurora (1313161554)
- BNB Chain (56)
- Avalanche (43114)
- Fantom (250)
- Moonbeam (1284)
- Lisk (201)
- Somnia (2332)
- And more...

## Security Features

### User Wallets

1. **Private Key Storage:**
   - EOA private key stored encrypted in database
   - Used to control smart wallet
   - Never exposed to frontend

2. **Smart Contract Security:**
   - Smart wallet is a contract (not EOA)
   - Can implement additional security features
   - Recovery mechanisms possible

3. **Gasless Transactions:**
   - Reduces phishing risk (no gas approval needed)
   - Better UX
   - Platform controls gas sponsorship

### Platform Wallets

1. **Multi-signature:**
   - 2-of-3 signature requirement
   - Enhanced security for platform operations
   - Prevents single point of failure

2. **Key Management:**
   - Three separate private keys
   - Stored securely in environment variables
   - Offline backup recommended

3. **Transaction Security:**
   - All transactions require multiple signatures
   - Queuing and confirmation system
   - Recovery procedures in place

## Wallet Initialization

### User Wallet Initialization

**Automatic:**
- Created during user registration
- No manual intervention needed
- Works immediately after creation

**Script:** `npm run init-wallets` (for platform wallets only)

### Platform Wallet Initialization

**Manual Process:**

1. **Set Environment Variables:**
   ```env
   PLATFORM_WALLET_PRIMARY_KEY=...
   PLATFORM_WALLET_SECONDARY_KEY=...
   PLATFORM_WALLET_BACKUP_KEY=...
   SMART_WALLET_FACTORY_ADDRESS=... (optional)
   ```

2. **Run Initialization:**
   ```bash
   npm run init-wallets
   ```

3. **Verify:**
   ```bash
   npm run check-balances
   ```

## ThirdWeb Client Configuration

**File:** `backend/src/services/auth.ts`

```typescript
import { createThirdwebClient } from "thirdweb";

export const client: ThirdwebClient = createThirdwebClient({
    secretKey: config.THIRDWEB_SECRET_KEY as string,
});
```

**Required Environment Variable:**
```env
THIRDWEB_SECRET_KEY=your_thirdweb_secret_key
```

Get your secret key from: https://thirdweb.com/dashboard

## Benefits of Account Abstraction

### 1. Gasless Transactions
- Users don't need native tokens for gas
- Better UX, especially for new users
- Platform can sponsor transactions

### 2. Unified Addresses
- Same wallet address on all chains
- Easier user experience
- Simplified address management

### 3. Enhanced Security
- Smart contract wallets can implement additional security
- Recovery mechanisms
- Multi-signature support

### 4. Batch Transactions
- Multiple operations in one transaction
- Lower gas costs
- Better efficiency

### 5. Social Recovery
- Can implement social recovery
- Key rotation capabilities
- Enhanced user security

## Code Examples

### Creating a User Wallet

```typescript
import { createAccount } from './services/auth';

// During registration
const { pk, walletAddress } = await createAccount();
// pk = EOA private key
// walletAddress = Smart wallet address (unified across chains)
```

### Using a Smart Wallet for Transfers

```typescript
import { smartWallet, privateKeyToAccount } from "thirdweb/wallets";
import { transfer } from "thirdweb/extensions/erc20";

// Create personal account
const personalAccount = privateKeyToAccount({
    client,
    privateKey: userPrivateKey
});

// Connect smart wallet
const wallet = smartWallet({
    chain: defineChain(42161), // Arbitrum
    sponsorGas: true,
});

const smartAccount = await wallet.connect({
    client,
    personalAccount,
});

// Execute transfer (gasless)
const tx = await sendTransaction({
    transaction: transfer({
        contract,
        to: recipientAddress,
        amount: "100.0",
    }),
    account: smartAccount,
});
```

### Platform Wallet Multi-sig

```typescript
// Initialize with 3 owners
const ownerKeys = [
    process.env.PLATFORM_WALLET_PRIMARY_KEY,
    process.env.PLATFORM_WALLET_SECONDARY_KEY,
    process.env.PLATFORM_WALLET_BACKUP_KEY,
];

const { address } = await initializePlatformWallet(ownerKeys, 'arbitrum');
// Returns smart wallet address requiring 2-of-3 signatures
```

## Summary

**Account Abstraction Framework:** ThirdWeb Smart Wallets (ERC-4337)

**Wallet Types:**
1. **User Wallets:** Single EOA-controlled smart wallets, gasless, unified addresses
2. **Platform Wallets:** 2-of-3 multisig smart wallets, configurable gas, per-chain or unified

**Key Features:**
- Gasless transactions (sponsored)
- Unified addresses across chains
- Smart contract wallets (not EOA)
- Multi-signature support for platform
- 18+ chain support

**Wallet Creation:**
- User wallets: Automatic during registration
- Platform wallets: Manual initialization with 3 keys

**Factory:**
- Uses ThirdWeb's default factory or custom `SMART_WALLET_FACTORY_ADDRESS`

The system leverages modern account abstraction to provide a seamless, gasless experience for users while maintaining security through smart contract wallets and multi-signature for platform operations.

