# NexusPay Frontend Analysis

## Overview

The NexusPay frontend is a **Next.js 14** application built with **React 18** and **TypeScript**. It's a Progressive Web App (PWA) that provides a complete user interface for the NexusPay crypto payment platform, connecting seamlessly to the backend API.

## Technology Stack

### Core Framework
- **Next.js 14.1.1** - React framework with App Router
- **React 18** - UI library
- **TypeScript 5.4.5** - Type safety

### UI & Styling
- **Tailwind CSS 3.3.0** - Utility-first CSS framework
- **Radix UI** - Accessible component primitives
  - Dialog, Dropdown Menu, Select, Toast, Slot
- **Lucide React** - Icon library
- **Phosphor Icons** - Additional icons
- **Font Awesome** - Icon library
- **GSAP 3.12.5** - Animation library

### State Management & Data Fetching
- **React Context API** - Global state (Auth, Wallet, Balance, Business, Chain, PWA)
- **@tanstack/react-query 5.34.1** - Server state management
- **Axios 1.6.8** - HTTP client

### Forms & Validation
- **React Hook Form 7.50.1** - Form management
- **Formik 2.4.6** - Alternative form library
- **Yup 1.4.0** - Schema validation

### PWA Features
- **@ducanh2912/next-pwa 10.2.6** - PWA support
- Service worker with offline support
- App manifest for installability

### Additional Libraries
- **react-hot-toast 2.4.1** - Toast notifications
- **qrcode.react 3.1.0** - QR code generation
- **react-otp-input 3.1.1** - OTP input component
- **next-themes 0.3.0** - Theme management
- **@reclaimprotocol/js-sdk** - Reclaim protocol integration

## Architecture

### Directory Structure

```
nexuspayapp/
├── app/                    # Next.js App Router pages
│   ├── auth/              # Authentication pages
│   ├── business/          # Business account pages
│   ├── crypto/            # Crypto operations
│   ├── dashboard/         # User dashboard
│   ├── mpesa/             # M-Pesa operations
│   ├── transactions/      # Transaction history
│   └── ...
├── components/            # React components
│   ├── auth/              # Authentication components
│   ├── business/          # Business components
│   ├── crypto/            # Crypto operation components
│   ├── dashboard/         # Dashboard components
│   ├── mpesa/             # M-Pesa components
│   ├── transactions/      # Transaction components
│   └── ui/                # Reusable UI components
├── context/                # React Context providers
│   ├── AuthContext.tsx    # Authentication state
│   ├── BalanceContext.tsx # Balance state
│   ├── BusinessContext.tsx # Business state
│   ├── ChainContext.tsx   # Blockchain chain state
│   ├── PWAContext.tsx     # PWA state
│   └── WalletContext.tsx  # Wallet state
├── hooks/                 # Custom React hooks
│   ├── useApi.ts          # Generic API hook
│   ├── useAxios.tsx       # Axios hook
│   ├── useCrypto.ts       # Crypto operations hook
│   ├── useMpesa.ts        # M-Pesa operations hook
│   ├── useBusiness.ts     # Business operations hook
│   └── ...
├── lib/                   # Core API integration
│   ├── api.ts             # Axios client with interceptors
│   ├── auth.ts            # Authentication API
│   ├── crypto.ts          # Crypto operations API
│   ├── mpesa.ts           # M-Pesa operations API
│   ├── business.ts        # Business API
│   ├── config.ts          # API configuration
│   └── ...
├── types/                 # TypeScript type definitions
│   ├── api-types.ts       # API response types
│   ├── form-types.ts      # Form data types
│   └── transaction-types.ts # Transaction types
└── public/                # Static assets
```

## Backend Integration

### API Configuration

The frontend automatically detects the environment and connects to the appropriate backend:

**Configuration (`lib/config.ts`):**
```typescript
// Development: http://localhost:8000/api
// Production: https://api.nexuspaydefi.xyz/api
```

**Auto-detection logic:**
- If hostname is `localhost`, `127.0.0.1`, or local network IP → uses `localhost:8000`
- Otherwise → uses production API `https://api.nexuspaydefi.xyz/api`
- Automatic fallback to production if localhost fails

### API Client Setup

**Axios Instance (`lib/api.ts`):**
- Base URL: Auto-detected based on environment
- Timeout: 30 seconds
- Headers: `Content-Type: application/json`
- Request interceptor: Automatically adds JWT token from localStorage
- Response interceptor: Handles 401 errors, token refresh, network fallback

**Token Management:**
- Stored in `localStorage` as `nexuspay_token`
- Automatically added to all requests as `Authorization: Bearer <token>`
- Token validation on app initialization
- Automatic logout on token expiration
- Token refresh mechanism for expired tokens

### Authentication Flow

**Registration:**
1. User submits registration form
2. `POST /api/auth/register/initiate` - Creates account, sends OTP
3. User enters OTP
4. `POST /api/auth/register/verify/email` or `/verify/phone` - Verifies OTP
5. JWT token returned and stored
6. User automatically logged in

**Login:**
1. User enters email/phone + password
2. `POST /api/auth/login` - Validates credentials, sends OTP
3. User enters OTP
4. `POST /api/auth/login/verify` - Verifies OTP, returns JWT token
5. User logged in

**Token Handling:**
- Token stored in `localStorage` and `sessionStorage`
- Multiple storage keys checked for backward compatibility
- Token format validation (JWT structure)
- Expiration checking
- Automatic cleanup on logout

## Key Features

### 1. Authentication System

**Components:**
- `LoginForm.tsx` - Login form with email/phone support
- `RegisterForm.tsx` - Registration form
- `GoogleSignIn.tsx` - Google OAuth integration
- `AuthGuard.tsx` - Route protection component

**Context:**
- `AuthContext.tsx` - Manages authentication state
- Provides: `user`, `loading`, `isAuthenticated`, `login()`, `register()`, `logout()`

**API Integration:**
- `lib/auth.ts` - All authentication API calls
- Supports email, phone, and Google OAuth
- OTP verification flow
- Password reset functionality

### 2. Crypto Operations

**Send Tokens:**
- Send crypto to email, phone, or wallet address
- Multi-chain support (Arbitrum, Polygon, Celo, etc.)
- Security verification (password or Google Auth)
- Real-time transaction tracking

**Pay Merchants:**
- Pay registered businesses with crypto
- Google Authenticator support
- Transaction confirmation flow

**Balance Management:**
- Real-time balance fetching
- Multi-chain balance display
- Balance validation before transactions

**Components:**
- `SendTokenForm.tsx` - Send crypto form
- `PayMerchantForm.tsx` - Pay business form
- `ReceiveInfo.tsx` - Receive crypto info with QR code
- `QRCodeScanner.tsx` - Scan QR codes

**Hooks:**
- `useCrypto()` - Crypto operations hook
- `useWalletBalance()` - Balance fetching hook
- `useBalanceValidation()` - Balance validation

### 3. M-Pesa Integration

**Buy Crypto (Fiat → Crypto):**
- M-Pesa STK Push integration
- Real-time transaction status
- Automatic crypto release on payment

**Sell Crypto (Crypto → Fiat):**
- Withdraw crypto to M-Pesa
- B2C payment integration
- Transaction tracking

**Pay with Crypto:**
- Revolutionary feature: Pay bills/shops with crypto
- Automatic crypto-to-KES conversion
- Support for Paybill and Till numbers
- Rollback protection if payment fails

**Components:**
- `BuyCryptoForm.tsx` - Buy crypto with M-Pesa
- `SendCryptoToMpesaForm.tsx` - Sell crypto to M-Pesa
- `PayBillForm.tsx` - Pay bills with crypto

**Hooks:**
- `useMpesa()` - M-Pesa operations hook
- Handles deposit, withdraw, buy crypto, pay with crypto

### 4. Business Accounts

**Features:**
- Business account creation
- Business dashboard
- Business analytics
- Business credit/overdraft
- Business QR codes for payments
- Business settings

**Components:**
- `BusinessDashboard.tsx` - Main business dashboard
- `BusinessCreationForm.tsx` - Create business account
- `BusinessAnalytics.tsx` - Business analytics
- `BusinessQRCode.tsx` - Business payment QR code
- `BusinessOverdraft.tsx` - Overdraft management

**Context:**
- `BusinessContext.tsx` - Business state management

**Hooks:**
- `useBusiness()` - Business operations
- `useBusinessFinance()` - Business finance operations

### 5. Transaction Management

**Features:**
- Transaction history with filters
- Real-time transaction status
- Multi-currency display (USD, KES)
- Blockchain explorer links
- Transaction details view

**Components:**
- `TransactionHistory.tsx` - Transaction list
- `TransactionCard.tsx` - Transaction card component
- `TransactionFilters.tsx` - Filter transactions

**Hooks:**
- `useTransactionHistory()` - Fetch transaction history

### 6. Dashboard

**Features:**
- User dashboard with stats
- Balance overview
- Recent transactions
- Quick actions
- Portfolio overview

**Components:**
- `DashboardCard.tsx` - Dashboard card component
- `StatTable.tsx` - Statistics table
- `ProgressBar.tsx` - Progress indicator

**Hooks:**
- `useDashboardData()` - Dashboard data fetching

### 7. PWA Features

**Progressive Web App:**
- Service worker for offline support
- App manifest for installability
- Offline page (`/offline`)
- Caching strategy:
  - API calls: Network-first with 24h cache
  - Images: Cache-first with 30d cache
  - Static assets: Stale-while-revalidate

**Context:**
- `PWAContext.tsx` - PWA state management

## State Management

### Context Providers

**AuthContext:**
- User authentication state
- Login/logout functions
- Registration flow
- Token management

**BalanceContext:**
- User balance state
- Multi-chain balances
- Balance updates

**BusinessContext:**
- Business account state
- Business operations

**ChainContext:**
- Selected blockchain chain
- Chain switching

**WalletContext:**
- Wallet information
- Wallet operations

**PWAContext:**
- PWA installation state
- Offline status

### React Query

Used for server state management:
- Automatic caching
- Background refetching
- Optimistic updates
- Error retry logic

## API Integration Patterns

### 1. API Client Pattern

**Base Setup (`lib/api.ts`):**
```typescript
const apiClient = axios.create({
  baseURL: getApiBaseUrl(),
  timeout: 30000,
  headers: { 'Content-Type': 'application/json' }
});
```

**Request Interceptor:**
- Automatically adds JWT token
- Handles token format variations
- Logs auth status for debugging

**Response Interceptor:**
- Handles 401/403 errors
- Automatic token refresh
- Network error fallback
- Error logging

### 2. API Module Pattern

Each domain has its own API module:

**Example (`lib/crypto.ts`):**
```typescript
export const cryptoAPI = {
  sendToken: async (data: SendTokenData) => {
    const response = await apiClient.post('/token/sendToken', data);
    return response.data;
  },
  // ... more methods
};
```

### 3. Hook Pattern

Custom hooks wrap API calls with state management:

**Example (`hooks/useCrypto.ts`):**
```typescript
export const useCrypto = () => {
  const sendTokenApi = useApi<SendTokenResponse>();
  
  const sendToken = useCallback(async (data: SendTokenData) => {
    return sendTokenApi.execute(
      () => cryptoAPI.sendToken(data),
      {
        showSuccessToast: false,
        showErrorToast: false,
      }
    );
  }, [sendTokenApi]);
  
  return { sendToken, sendTokenLoading: sendTokenApi.loading };
};
```

### 4. Generic API Hook

**`useApi` hook (`hooks/useApi.ts`):**
- Manages loading state
- Handles errors
- Shows toast notifications
- Provides success/error callbacks

## Error Handling

### Standardized Error Responses

All API errors follow this format:
```typescript
{
  success: false,
  message: "Human-readable error message",
  error: {
    code: "ERROR_CODE",
    message: "Detailed error message"
  }
}
```

### Error Handling Strategy

1. **API Level** (`lib/api.ts`):
   - Network errors → fallback to production
   - 401 errors → token refresh or logout
   - 500 errors → detailed logging

2. **Hook Level** (`hooks/useApi.ts`):
   - Automatic error toast notifications
   - Error state management
   - Custom error callbacks

3. **Component Level**:
   - Try-catch blocks for critical operations
   - User-friendly error messages
   - Error boundaries for React errors

## Routing Structure

### App Router Pages

**Authentication:**
- `/login` - Login page
- `/signup` - Registration page
- `/forgotpassword` - Password reset

**Main Features:**
- `/dashboard` - User dashboard
- `/home` - Home page
- `/onboarding` - Onboarding flow

**Crypto Operations:**
- `/send` - Send crypto
- `/receive` - Receive crypto (QR code)
- `/crypto` - Crypto operations hub

**M-Pesa Operations:**
- `/buy-crypto` - Buy crypto with M-Pesa
- `/pay` - Pay with crypto

**Business:**
- `/business` - Business dashboard
- `/business/create` - Create business account
- `/business/overdraft` - Business overdraft

**Transactions:**
- `/transactions` - Transaction history

**Settings:**
- `/settings` - User settings

## Security Features

### Authentication Security

1. **JWT Token Management:**
   - Secure token storage
   - Token validation
   - Automatic expiration handling
   - Token refresh mechanism

2. **Request Security:**
   - All authenticated requests include JWT token
   - Automatic token injection
   - Token format validation

3. **Password Security:**
   - Password input masking
   - Password strength validation
   - Secure password reset flow

### Transaction Security

1. **Two-Factor Authentication:**
   - OTP verification for login
   - Google Authenticator support
   - Password confirmation for transactions

2. **Balance Validation:**
   - Pre-transaction balance checks
   - Insufficient balance error handling
   - Real-time balance updates

## Performance Optimizations

### 1. Code Splitting
- Next.js automatic code splitting
- Dynamic imports for heavy components
- Route-based code splitting

### 2. Caching
- React Query caching
- PWA service worker caching
- API response caching (24h for API calls)

### 3. Image Optimization
- Next.js Image component
- WebP/AVIF format support
- Lazy loading
- 30-day image cache

### 4. Bundle Optimization
- Tree shaking
- Package import optimization
- SWC minification

## Testing Approach

### Component Testing
- React components can be tested with React Testing Library
- Example components in `components/` directory

### API Testing
- Direct API calls using `apiClient`
- Mock API responses for testing
- Error scenario testing

### Integration Testing
- Full user flows (registration → login → transaction)
- End-to-end testing with backend

## Development Workflow

### Starting Development

1. **Start Backend:**
   ```bash
   cd backend
   npm run dev
   ```

2. **Start Frontend:**
   ```bash
   cd nexuspayapp
   npm run dev
   ```

3. **Access:**
   - Frontend: http://localhost:3000
   - Backend: http://localhost:8000

### Environment Configuration

**Development:**
- Automatically uses `localhost:8000` for API
- Hot reload enabled
- Detailed error messages

**Production:**
- Uses `https://api.nexuspaydefi.xyz/api`
- Optimized builds
- Error monitoring

## Key Integration Points

### 1. Authentication Endpoints

**Frontend → Backend:**
- `POST /api/auth/register/initiate`
- `POST /api/auth/register/verify/email`
- `POST /api/auth/login`
- `POST /api/auth/login/verify`
- `GET /api/auth/me`

### 2. Crypto Endpoints

**Frontend → Backend:**
- `POST /api/token/sendToken`
- `POST /api/token/pay`
- `GET /api/token/balance`
- `GET /api/token/receive`

### 3. M-Pesa Endpoints

**Frontend → Backend:**
- `POST /api/mpesa/deposit`
- `POST /api/mpesa/buy-crypto`
- `POST /api/mpesa/withdraw`
- `POST /api/mpesa/pay-with-crypto`
- `GET /api/mpesa/transaction-status`

### 4. Transaction Endpoints

**Frontend → Backend:**
- `GET /api/transactions/history`
- `GET /api/transactions/:id`

### 5. Business Endpoints

**Frontend → Backend:**
- `POST /api/business/create`
- `GET /api/business/:id`
- `GET /api/business/:id/analytics`

## Notable Features

### 1. Automatic API Detection
- Detects localhost vs production
- Automatic fallback to production if localhost fails
- No manual configuration needed

### 2. Comprehensive Error Handling
- Network error fallback
- Token refresh on expiration
- User-friendly error messages
- Detailed error logging

### 3. Real-time Updates
- Balance updates
- Transaction status polling
- Real-time notifications

### 4. PWA Support
- Offline functionality
- Installable app
- Service worker caching
- Offline page

### 5. Multi-chain Support
- Support for 18+ blockchain networks
- Chain switching
- Multi-chain balance display

## Production Deployment

### Build Process

```bash
npm run build
npm start
```

### Environment Variables

```env
NEXT_PUBLIC_API_BASE_URL=https://api.nexuspaydefi.xyz/api
```

### Vercel Deployment

- Configured with `vercel.json`
- Automatic deployments
- Environment variable management
- Analytics integration

## Summary

The NexusPay frontend is a **production-ready, feature-rich** Next.js application that:

1. **Seamlessly integrates** with the backend API
2. **Provides comprehensive** crypto payment features
3. **Offers excellent UX** with PWA support
4. **Implements robust** error handling and security
5. **Uses modern** React patterns and TypeScript
6. **Supports multiple** blockchain networks
7. **Includes business** account features
8. **Offers revolutionary** "Pay with Crypto" functionality

The architecture is **modular, scalable, and maintainable**, making it easy to add new features and maintain the codebase.

