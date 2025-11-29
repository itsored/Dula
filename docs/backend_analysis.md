# NexusPay Backend Analysis

## Overview

The NexusPay backend is a comprehensive **crypto payment platform** built with **Node.js/TypeScript** and **Express.js**. It integrates M-Pesa (Kenya's mobile money service) with blockchain networks to enable seamless fiat-to-crypto and crypto-to-fiat transactions.

## Core Technology Stack

- **Runtime**: Node.js (v18+)
- **Framework**: Express.js
- **Language**: TypeScript
- **Database**: MongoDB (via Mongoose)
- **Cache**: Redis (ioredis)
- **Blockchain**: ThirdWeb SDK, Ethers.js, Viem
- **Authentication**: JWT tokens
- **Payment Integration**: M-Pesa API (Safaricom)
- **Communication**: 
  - Email (Nodemailer)
  - SMS (Africa's Talking API)
  - Push Notifications (Push Protocol)

## Architecture

### Layered Architecture

The backend follows a **layered architecture** pattern:

```
Frontend (Next.js)
    ↓
API Gateway (Express + Security Middleware)
    ↓
Route Layer (10 route modules)
    ↓
Middleware Layer (Auth, Validation, Rate Limiting)
    ↓
Controller Layer (11 controllers)
    ↓
Service Layer (25+ services)
    ↓
Data Layer (MongoDB + Redis)
    ↓
External Integrations (Blockchain + Payment APIs)
```

### Key Directories

#### `/src/routes/` - API Endpoints
- `authRoutes.ts` - Authentication & user management
- `businessRoutes.ts` - Business account operations
- `tokenRoutes.ts` - Crypto token operations
- `mpesaRoutes.ts` - M-Pesa payment integration
- `adminRoutes.ts` - Administrative functions
- `transactionRoutes.ts` - Transaction history & analytics
- `liquidityRoutes.ts` - Liquidity provision
- `rampRoutes.ts` - Fiat on/off ramp
- `platformWalletRoutes.ts` - Platform wallet management
- `usdcRoutes.ts` - USDC-specific operations
- `yieldRoutes.ts` - Yield farming operations

#### `/src/controllers/` - Business Logic Handlers
- `authController.ts` - User authentication & registration
- `mpesaController.ts` - M-Pesa transaction handling (largest file, ~3195 lines)
- `tokenController.ts` - Token operations
- `transactionController.ts` - Transaction management
- `businessController.ts` - Business account management
- `adminController.ts` - Admin operations
- `liquidityController.ts` - Liquidity management
- `rampController.ts` - Ramp operations
- `platformWalletController.ts` - Platform wallet operations
- `monitoringController.ts` - System monitoring
- `usdcController.ts` - USDC operations
- `yieldController.ts` - Yield operations

#### `/src/services/` - Core Services
**Infrastructure Services:**
- `platformWallet.ts` (82KB) - Multi-chain wallet management
- `database.ts` - MongoDB connection & management
- `redis.ts` - Redis cache management
- `scheduler.ts` - Background job scheduling
- `queue.ts` - Transaction queue management

**Financial Services:**
- `mpesa.ts` (30KB) - M-Pesa API integration
- `token.ts` (26KB) - Token operations
- `rates.ts` (7.9KB) - Exchange rate management
- `feeService.ts` (11KB) - Fee calculation
- `swapService.ts` - Token swapping
- `rampService.ts` (8KB) - Fiat ramp operations
- `liquidityService.ts` (16KB) - Liquidity management
- `yieldService.ts` - Yield farming

**Transaction Services:**
- `transactionLogger.ts` (6.8KB) - Transaction logging
- `transactionMonitor.ts` (12KB) - Transaction monitoring
- `transactionRecovery.ts` (7.8KB) - Transaction recovery
- `transactionVerification.ts` - Transaction verification
- `transactionRecorder.ts` - Transaction recording
- `reconciliation.ts` - Financial reconciliation
- `cryptoRelease.ts` - Crypto release management

**Communication Services:**
- `email.ts` (10KB) - Email service
- `smsService.ts` - SMS service (Africa's Talking)
- `otpService.ts` - OTP generation & validation

**Other Services:**
- `auth.ts` (3.8KB) - Authentication utilities
- `googleAuth.ts` - Google OAuth integration
- `wallet.ts` - Wallet operations
- `utils.ts` (3.5KB) - Utility functions
- `mpesaUtils.ts` - M-Pesa utilities
- `mpesaRetry.ts` - M-Pesa retry logic
- `businessCreditService.ts` - Business credit management
- `userOptimizationService.ts` - User optimization
- `liquidityUsageTracker.ts` - Liquidity tracking

#### `/src/models/` - Data Models
- `userModel.ts` - User schema
- `businessModel.ts` - Business account schema
- `escrowModel.ts` - Escrow transaction schema
- `verificationModel.ts` - Verification records
- `RampTransaction.ts` - Ramp transaction schema
- `LiquidityProvider.ts` - Liquidity provider schema

#### `/src/middleware/` - Middleware
- `auth.ts` - JWT authentication
- `authMiddleware.ts` - Additional auth middleware
- `strictAuthMiddleware.ts` - Strict authentication
- `roleMiddleware.ts` - Role-based access control
- `validation.ts` - Request validation
- `rateLimiting.ts` - Rate limiting
- `/validators/` - Validation schemas (Zod, Express Validator)

#### `/src/config/` - Configuration
- `env.ts` - Environment configuration (development/production/test)
- `tokens.ts` - Token configurations
- `constants.ts` - System constants
- `platformWallet.ts` - Platform wallet config
- `redis.ts` - Redis configuration
- `logger.ts` - Logging configuration
- `abi.ts` - Smart contract ABIs

## Key Features

### 1. Authentication & User Management
- **Registration**: Email/phone verification with OTP
- **Login**: JWT-based authentication with OTP verification
- **Password Reset**: Email and phone-based password reset
- **Google OAuth**: Google account linking
- **Account Deletion**: Secure account deletion flow
- **Session Management**: Redis-based session storage

### 2. M-Pesa Integration
- **STK Push**: Initiate M-Pesa payments (deposits)
- **B2C Payments**: Business-to-consumer payments (withdrawals)
- **B2B Payments**: Business-to-business payments
- **Webhook Handling**: STK, B2C, B2B, and queue timeout callbacks
- **Transaction Retry**: Automatic retry for failed transactions
- **Receipt Verification**: Manual receipt submission
- **Transaction Status**: Real-time transaction status checking

### 3. Multi-Chain Support
The platform supports **18+ blockchain networks**:

**EVM Chains:**
- Arbitrum (42161) - USDC: `0xaf88d065e77c8cC2239327C5EDb3A432268e5831`
- Polygon (137) - USDC: `0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359`
- Base (8453) - USDC: `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`
- Optimism (10) - USDC: `0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85`
- Celo (42220) - USDC: `0x48065fbBE25f71C9282ddf5e1cD6D6A887483D5e`
- Scroll (534352) - USDC: `0x06eFdBFf2a14a7c8E15944D1F4A48F9F95F663A4`
- Fuse (122) - USDC: `0x620fd5fa44BE6af63715Ef4E65DDFA0387aD13F5`
- Gnosis (100) - USDC: `0xDDAfbb505ad214D7b80b1f830fcCc89B60fb7A83`
- Aurora (1313161554) - USDC: `0xB12BFcA5A55806AaF64E99521918A4bf0fC40802`
- BNB Chain (56) - USDC: `0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d`
- Avalanche (43114) - USDC: `0xB97EF9Ef8734C71904D8002F8b6Bc66Dd9c48a6E`
- Fantom (250) - USDC: `0x04068DA6C83AFCFA0e13ba15A6696662335D5B75`
- Moonbeam (1284) - USDC: `0x818ec0A7Fe18Ff94269904fCED6AE3DaE6d6dC0b`
- Lisk (201) - USDC: `0x4e05F8C19EaA61520a94850dC41EAc3c39927696`
- Somnia (2332) - USDC: `0x1C7312Cb60b40cF586e796FEdD60Cf243286c9E9`

**Supported Tokens:**
- USDC (primary stablecoin across all chains)
- USDT (on Arbitrum, Polygon, Optimism)
- DAI (on Arbitrum, Polygon, Optimism)
- Native tokens (ETH, CELO, etc.)

### 4. Platform Wallet System
- **Multi-Signature Wallets**: 2-of-3 signature requirement for security
- **Multi-Chain Wallets**: Separate wallet addresses per chain
- **Smart Wallet Factory**: ThirdWeb smart wallet integration
- **Balance Management**: Real-time balance tracking
- **Transaction Signing**: Secure transaction signing with multiple keys
- **Gas Management**: Gas price optimization and caching

### 5. Transaction Management
- **Transaction Types**:
  - `fiat_to_crypto` - M-Pesa to crypto
  - `crypto_to_fiat` - Crypto to M-Pesa
  - `crypto_to_paybill` - Crypto to M-Pesa paybill
  - `crypto_to_till` - Crypto to M-Pesa till
  - `token_transfer` - Crypto to crypto transfers
- **Transaction States**: pending, completed, failed, error, reserved
- **Transaction History**: Filtered history with pagination
- **On-chain Verification**: Blockchain transaction verification
- **Transaction Recovery**: Automatic recovery for failed transactions
- **Transaction Monitoring**: Real-time transaction status monitoring

### 6. Liquidity Management
- **Liquidity Providers**: Track liquidity across chains
- **Liquidity Checking**: Real-time liquidity status
- **Multi-Chain Overview**: Cross-chain liquidity visibility
- **Liquidity Usage Tracking**: Monitor liquidity consumption

### 7. Ramp Operations
- **Fiat On-Ramp**: M-Pesa to crypto conversion
- **Fiat Off-Ramp**: Crypto to M-Pesa conversion
- **Rate Management**: Dynamic exchange rates
- **Fee Calculation**: Transparent fee structure

### 8. Business Features
- **Business Accounts**: Separate business account management
- **Business Wallets**: Business-specific wallet management
- **Business Credits**: Credit management for businesses
- **Business-to-Personal Transfers**: B2P transfer capabilities

### 9. Admin Features
- **User Management**: List, view, promote users
- **Transaction Management**: View and update transactions
- **Wallet Management**: Platform wallet balance checking
- **Fee Management**: Withdraw fees to main wallet
- **User Funding**: Fund user wallets (admin only)

### 10. Security Features
- **Rate Limiting**: 100 requests per 15 minutes per IP
- **CORS Protection**: Configurable allowed origins
- **Helmet**: Security headers
- **Input Validation**: Comprehensive validation (Zod, Express Validator)
- **JWT Authentication**: Secure token-based auth
- **Role-Based Access Control**: Admin/user role separation
- **Strict Authentication**: Enhanced auth for sensitive operations

### 11. Monitoring & Recovery
- **Transaction Monitoring**: Real-time transaction status
- **Transaction Recovery**: Automatic retry for failed transactions
- **Scheduler**: Background job processing
- **Queue Management**: Transaction queue with retry logic
- **Reconciliation**: Financial reconciliation system
- **Logging**: Comprehensive transaction logging

### 12. Exchange Rates
- **Real-time Rates**: Dynamic rate fetching
- **Rate Caching**: Redis-based rate caching
- **Multi-Source**: Support for multiple rate providers
- **Conversion**: Fiat to crypto and vice versa

## API Structure

### Base URL
```
Development: http://localhost:8000/api
Production: https://api.nexuspay.app/api
```

### Standard Response Format
```json
{
  "success": true/false,
  "message": "Human-readable message",
  "data": { ... },
  "error": {
    "code": "ERROR_CODE",
    "message": "Error details"
  },
  "timestamp": "ISO timestamp"
}
```

### Authentication
All protected endpoints require:
```
Authorization: Bearer <JWT_TOKEN>
```

## Environment Configuration

The system supports three environments:
- **Development**: Uses dev M-Pesa credentials, dev MongoDB
- **Production**: Uses production M-Pesa credentials, prod MongoDB
- **Test**: Uses sandbox M-Pesa, test MongoDB

### Key Environment Variables
- `JWT_SECRET` - JWT signing secret
- `THIRDWEB_SECRET_KEY` - ThirdWeb API key
- `MONGO_URL` - MongoDB connection string
- `REDIS_URL` - Redis connection string
- `MPESA_*_CONSUMER_KEY` - M-Pesa API credentials
- `MPESA_*_CONSUMER_SECRET` - M-Pesa API credentials
- `MPESA_*_SHORTCODE` - M-Pesa business shortcode
- `PLATFORM_WALLET_PRIVATE_KEY` - Platform wallet private key
- `EMAIL_USER`, `EMAIL_APP_PASSWORD` - Email credentials
- `AFRICAS_TALKING_API_KEY` - SMS service API key
- Chain-specific API keys for blockchain explorers

## Database Schema

### Collections
1. **users** - User accounts
   - email, phoneNumber, password, walletAddress
   - role (user/admin), status (active/inactive/suspended)

2. **businesses** - Business accounts
   - Business-specific fields and settings

3. **transactions** - Transaction records
   - Transaction type, status, amounts, chains, tokens

4. **escrows** - Escrow transactions
   - Pending transaction management

5. **verifications** - Verification records
   - Email/phone verification tracking

6. **liquidityprovisions** - Liquidity provider data

7. **ramptransactions** - Ramp transaction records

## Key Workflows

### 1. User Registration Flow
1. User submits registration (email/phone)
2. OTP sent via email/SMS
3. User verifies OTP
4. Wallet address generated (ThirdWeb)
5. User account created
6. JWT token returned

### 2. M-Pesa Deposit Flow (Fiat → Crypto)
1. User initiates deposit request
2. STK Push sent to user's phone
3. User completes M-Pesa payment
4. Webhook received from M-Pesa
5. Crypto released to user's wallet
6. Transaction recorded

### 3. M-Pesa Withdrawal Flow (Crypto → Fiat)
1. User initiates withdrawal request
2. Crypto locked in escrow
3. B2C payment initiated to user's phone
4. Webhook confirms payment
5. Escrow released
6. Transaction completed

### 4. Transaction Recovery Flow
1. Scheduler checks for pending transactions
2. Failed transactions identified
3. Retry logic applied
4. Status updated based on result
5. Manual intervention flagged if needed

## Testing

### Admin API Testing
```bash
npm run test:admin              # Test all admin endpoints
npm run test:admin:users        # Test user management
npm run test:admin:transactions # Test transaction management
npm run test:admin:wallets      # Test wallet management
```

### Postman Collection
- `nexuspay-admin-api.postman_collection.json` - Pre-configured API tests

## Deployment

### Vercel Configuration
- `vercel.json` - Vercel deployment config
- Serverless function support
- Environment variable management

### Scripts
- `npm run dev` - Development server (nodemon)
- `npm run build` - TypeScript compilation
- `npm run start` - Production server
- `npm run init-wallets` - Initialize platform wallets
- `npm run check-balances` - Check wallet balances
- `npm run monitor` - Monitor system health

## Documentation Files

The backend includes extensive documentation:
- `README.md` - Main documentation
- `API_DOCUMENTATION.md` - Complete API reference
- `NEXUSPAY_ARCHITECTURE_OVERVIEW.md` - Architecture details
- `NEXUSPAY_IMPLEMENTATION_PLAN.md` - Implementation roadmap
- `IMPLEMENTATION_CHECKLIST.md` - Progress tracking
- `MPESA_INTEGRATION.md` - M-Pesa integration guide
- Multiple endpoint-specific documentation files

## Notable Features

1. **Comprehensive Error Handling**: Standardized error responses across all endpoints
2. **Transaction Queue**: Queue-based transaction processing with retry logic
3. **Multi-Chain Support**: Seamless support for 18+ blockchain networks
4. **Real-time Monitoring**: Transaction status monitoring and recovery
5. **Security First**: Multiple layers of security (auth, validation, rate limiting)
6. **Scalable Architecture**: Service-oriented design for easy scaling
7. **Extensive Logging**: Comprehensive transaction and system logging
8. **Webhook Management**: Robust webhook handling for M-Pesa callbacks

## Code Statistics

- **Total Routes**: 10 route modules
- **Total Controllers**: 11 controllers
- **Total Services**: 25+ services
- **Largest Files**:
  - `platformWallet.ts`: ~82KB
  - `mpesaController.ts`: ~3195 lines
  - `mpesa.ts`: ~30KB
  - `token.ts`: ~26KB
  - `liquidityService.ts`: ~16KB
  - `transactionMonitor.ts`: ~12KB

## Integration Points

### External Services
1. **ThirdWeb**: Smart wallet creation and management
2. **M-Pesa API**: Payment processing
3. **Africa's Talking**: SMS services
4. **Blockchain Networks**: 18+ EVM-compatible chains
5. **Email Service**: SMTP email delivery
6. **Push Protocol**: Push notifications
7. **Blockchain Explorers**: Transaction verification (Arbiscan, Polygonscan, etc.)

## Future Enhancements (from README)

1. **WebSocket Server**: Real-time updates
2. **Enhanced Business Features**: Business verification, advanced wallet management
3. **Automated Testing**: Comprehensive test suite
4. **Performance Monitoring**: Advanced monitoring and tracking
5. **Additional Chains**: Support for more blockchain networks

