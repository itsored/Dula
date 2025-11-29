# NexusPay Backend Testing Guide

## Overview

The NexusPay backend is a comprehensive crypto payment platform API built with Node.js, TypeScript, and Express. This guide covers all methods to test the backend API.

## Prerequisites

### Required Software
- **Node.js** (v18.0.0 or higher)
- **MongoDB** (local or cloud instance)
- **Redis** (optional but recommended for caching and OTP storage)
- **npm** or **yarn** package manager

### Required Services & API Keys
- MongoDB connection string
- Redis URL (optional)
- ThirdWeb Secret Key (for blockchain operations)
- M-Pesa API credentials (for payment testing)
- Africa's Talking API key (for SMS/OTP)
- Email credentials (for email OTP)
- Google OAuth credentials (optional)

## Setup Instructions

### 1. Install Dependencies

```bash
cd backend
npm install
```

### 2. Environment Configuration

Copy the example environment file and configure it:

```bash
cp env.example .env
```

Edit `.env` with your actual credentials. **Minimum required variables:**

```env
# Core Configuration
PORT=8000
NODE_ENV=development

# Database
DEV_MONGO_URL=mongodb://localhost:27017/nexuspay
# or
DEV_MONGO_URL=mongodb+srv://username:password@cluster.mongodb.net/nexuspay

# Security
JWT_SECRET=your_secure_jwt_secret_minimum_32_characters_long

# Blockchain
THIRDWEB_SECRET_KEY=your_thirdweb_secret_key

# SMS Service
AFRICAS_TALKING_API_KEY=your_africas_talking_api_key

# M-Pesa (Development)
MPESA_DEV_CONSUMER_KEY=your_mpesa_consumer_key
MPESA_DEV_CONSUMER_SECRET=your_mpesa_consumer_secret
MPESA_DEV_SHORTCODE=your_mpesa_shortcode
MPESA_DEV_PASSKEY=your_mpesa_passkey
MPESA_DEV_B2C_SHORTCODE=your_b2c_shortcode
MPESA_DEV_INITIATOR_NAME=testapi
MPESA_DEV_SECURITY_CREDENTIAL=your_encrypted_security_credential

# Platform Wallet
DEV_PLATFORM_WALLET_PRIVATE_KEY=your_private_key
DEV_PLATFORM_WALLET_ADDRESS=your_wallet_address

# Optional but Recommended
REDIS_URL=redis://localhost:6379
EMAIL_USER=your_email@gmail.com
EMAIL_APP_PASSWORD=your_app_password
```

### 3. Build the Project

```bash
npm run build
```

This compiles TypeScript to JavaScript in the `dist/` folder.

### 4. Initialize Platform Wallets (Optional)

If testing platform wallet features:

```bash
npm run init-wallets
```

## Starting the Server

### Development Mode (with auto-reload)

```bash
npm run dev
```

This uses `nodemon` to automatically restart the server when files change.

### Production Mode

```bash
npm run build
npm start
```

The server will start on `http://localhost:8000` (or the PORT specified in `.env`).

### Verify Server is Running

Check the health endpoint:

```bash
curl http://localhost:8000/api/health
```

Expected response:
```json
{
  "success": true,
  "message": "Service is healthy",
  "data": {
    "uptime": 123.45,
    "timestamp": 1234567890123
  }
}
```

## Testing Methods

### Method 1: Command-Line Testing Script

The backend includes a built-in testing script for admin endpoints.

#### Build First (if not already built)

```bash
npm run build
```

#### Run Tests

```bash
# Test all admin endpoints (non-destructive)
npm run test:admin

# Test specific endpoint groups
npm run test:admin:users        # User management endpoints
npm run test:admin:transactions # Transaction endpoints
npm run test:admin:wallets      # Wallet endpoints

# Test specific endpoints with parameters
node dist/scripts/testAdminAPI.js -e users
node dist/scripts/testAdminAPI.js -e user -i <user_id>
node dist/scripts/testAdminAPI.js -e transactions
node dist/scripts/testAdminAPI.js -e transaction -i <transaction_id>
node dist/scripts/testAdminAPI.js -e wallets
```

**Note:** The test script will:
1. Connect to your database
2. Find or create a test user
3. Promote that user to admin
4. Generate a JWT token
5. Test the endpoints

### Method 2: Postman Collection

#### Import Collection

1. Open Postman
2. Click **Import** button
3. Select `nexuspay-admin-api.postman_collection.json` from the backend directory
4. The collection will be imported with all endpoints

#### Configure Environment

1. Create a new Postman Environment
2. Add these variables:
   - `base_url`: `http://localhost:8000/api`
   - `admin_token`: Your JWT token (get from login or test script)

#### Get Admin Token

You can get an admin token by:
1. Logging in through the API (see Authentication section)
2. Using the test script which generates one
3. Manually creating an admin user in the database

### Method 3: cURL Commands

#### Get Admin Token First

```bash
# Login to get token (replace with actual credentials)
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@example.com",
    "password": "password123"
  }'
```

Save the token from the response.

#### Test Admin Endpoints

```bash
# Set your token
TOKEN="your_jwt_token_here"

# Get all users
curl -X GET http://localhost:8000/api/admin/users \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"

# Get specific user
curl -X GET http://localhost:8000/api/admin/users/USER_ID \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"

# Promote user to admin
curl -X POST http://localhost:8000/api/admin/users/promote/USER_ID \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"

# Get all transactions
curl -X GET http://localhost:8000/api/admin/transactions \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"

# Get specific transaction
curl -X GET http://localhost:8000/api/admin/transactions/TRANSACTION_ID \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"

# Update transaction status
curl -X PUT http://localhost:8000/api/admin/transactions/TRANSACTION_ID/status \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "completed",
    "notes": "Manually approved"
  }'

# Check platform wallets
curl -X GET http://localhost:8000/api/admin/platform-wallets \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"

# Fund user wallet
curl -X POST http://localhost:8000/api/admin/wallets/fund \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "USER_ID",
    "amount": "100.00",
    "chain": "arbitrum",
    "tokenSymbol": "USDC"
  }'
```

### Method 4: Browser Console Testing

Open browser console and use:

```javascript
// Set your admin token
const token = 'your_admin_jwt_token_here';

// Function to make API requests
async function callAPI(endpoint, method = 'GET', body = null) {
  const url = `http://localhost:8000/api/${endpoint}`;
  const options = {
    method,
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  };
  
  if (body) {
    options.body = JSON.stringify(body);
  }
  
  const response = await fetch(url, options);
  return await response.json();
}

// Example calls
callAPI('admin/users').then(console.log);
callAPI('admin/transactions').then(console.log);
callAPI('admin/platform-wallets').then(console.log);
```

### Method 5: HTTPie (Alternative to cURL)

```bash
# Install HTTPie: pip install httpie

# Get users
http GET http://localhost:8000/api/admin/users \
  Authorization:"Bearer $TOKEN"

# Create transaction
http POST http://localhost:8000/api/admin/wallets/fund \
  Authorization:"Bearer $TOKEN" \
  userId="USER_ID" \
  amount="100.00" \
  chain="arbitrum" \
  tokenSymbol="USDC"
```

## Testing Different Endpoint Categories

### 1. Authentication Endpoints

**Base URL:** `/api/auth`

#### Register User
```bash
curl -X POST http://localhost:8000/api/auth/register/initiate \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "phoneNumber": "+254712345678",
    "password": "SecurePassword123",
    "verifyWith": "email"
  }'
```

#### Verify Email
```bash
curl -X POST http://localhost:8000/api/auth/register/verify/email \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "otp": "123456"
  }'
```

#### Login
```bash
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "SecurePassword123"
  }'
```

#### Get Current User
```bash
curl -X GET http://localhost:8000/api/auth/me \
  -H "Authorization: Bearer $TOKEN"
```

### 2. Token/Crypto Endpoints

**Base URL:** `/api/token`

#### Get User Balance
```bash
curl -X GET http://localhost:8000/api/token/balance \
  -H "Authorization: Bearer $TOKEN"
```

#### Get Balance by Chain
```bash
curl -X GET http://localhost:8000/api/token/balance/arbitrum \
  -H "Authorization: Bearer $TOKEN"
```

#### Send Token
```bash
curl -X POST http://localhost:8000/api/token/sendToken \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "recipientIdentifier": "user@example.com",
    "amount": 50.00,
    "senderAddress": "0x742d35Cc6634C0532925a3b8D4C9db96C4b4d8b6",
    "chain": "arbitrum",
    "tokenSymbol": "USDC",
    "password": "userPassword"
  }'
```

#### Pay Business
```bash
curl -X POST http://localhost:8000/api/token/pay \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "senderAddress": "0x742d35Cc6634C0532925a3b8D4C9db96C4b4d8b6",
    "merchantId": "MERCH123",
    "amount": 100.00,
    "confirm": true,
    "chainName": "polygon",
    "tokenSymbol": "USDT",
    "googleAuthCode": "123456"
  }'
```

### 3. M-Pesa Endpoints

**Base URL:** `/api/mpesa`

#### Deposit (Buy Crypto)
```bash
curl -X POST http://localhost:8000/api/mpesa/deposit \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 1000,
    "phoneNumber": "+254712345678",
    "chain": "arbitrum",
    "tokenSymbol": "USDC"
  }'
```

#### Withdraw (Sell Crypto)
```bash
curl -X POST http://localhost:8000/api/mpesa/withdraw \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50.00,
    "phoneNumber": "+254712345678",
    "chain": "arbitrum",
    "tokenSymbol": "USDC",
    "password": "userPassword"
  }'
```

#### Get Transaction Status
```bash
curl -X GET "http://localhost:8000/api/mpesa/transaction-status?transactionId=TX123" \
  -H "Authorization: Bearer $TOKEN"
```

### 4. Transaction Endpoints

**Base URL:** `/api/transactions`

#### Get Transaction History
```bash
curl -X GET "http://localhost:8000/api/transactions/history?page=1&limit=10" \
  -H "Authorization: Bearer $TOKEN"
```

#### Get Transaction by ID
```bash
curl -X GET http://localhost:8000/api/transactions/TRANSACTION_ID \
  -H "Authorization: Bearer $TOKEN"
```

### 5. Admin Endpoints

**Base URL:** `/api/admin` (requires admin role)

All admin endpoints require:
- Valid JWT token
- User must have `role: 'admin'`

See Method 3 (cURL) section above for examples.

### 6. Platform Wallet Endpoints

**Base URL:** `/api/platform-wallet`

#### Get Platform Wallet Status
```bash
curl -X GET http://localhost:8000/api/platform-wallet/status \
  -H "Authorization: Bearer $TOKEN"
```

## Testing Workflows

### Complete User Registration Flow

1. **Initiate Registration**
```bash
curl -X POST http://localhost:8000/api/auth/register/initiate \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "phoneNumber": "+254712345678",
    "password": "SecurePass123",
    "verifyWith": "email"
  }'
```

2. **Check Email for OTP** (or check Redis/console logs)

3. **Verify Email**
```bash
curl -X POST http://localhost:8000/api/auth/register/verify/email \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "otp": "123456"
  }'
```

4. **Save the JWT token from response**

5. **Test Authenticated Endpoint**
```bash
curl -X GET http://localhost:8000/api/auth/me \
  -H "Authorization: Bearer $TOKEN"
```

### Complete M-Pesa Deposit Flow

1. **Initiate Deposit**
```bash
curl -X POST http://localhost:8000/api/mpesa/deposit \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 1000,
    "phoneNumber": "+254712345678",
    "chain": "arbitrum",
    "tokenSymbol": "USDC"
  }'
```

2. **Complete M-Pesa Payment** on your phone (STK Push)

3. **Check Transaction Status**
```bash
curl -X GET "http://localhost:8000/api/mpesa/transaction-status?transactionId=TX_ID" \
  -H "Authorization: Bearer $TOKEN"
```

## Utility Scripts

### Check Wallet Balances

```bash
npm run check-balances
```

### Monitor System

```bash
npm run monitor
```

### Test ENS Integration

```bash
npm run test:ens
```

## Common Issues & Troubleshooting

### Issue: Server Won't Start

**Error:** `Error: connect ECONNREFUSED`

**Solutions:**
1. Check MongoDB is running: `mongod` or verify cloud connection
2. Verify MongoDB URL in `.env`
3. Check if port 8000 is already in use: `lsof -i :8000`
4. Change PORT in `.env` if needed

### Issue: Authentication Errors

**Error:** `Unauthorized: Admin access required`

**Solutions:**
1. Verify token is valid and not expired
2. Check user has `role: 'admin'` in database
3. Ensure token is in correct format: `Bearer <token>`
4. Regenerate token by logging in again

### Issue: Database Connection Failed

**Error:** `MongoServerError: connection failed`

**Solutions:**
1. Verify MongoDB connection string format
2. Check network connectivity
3. Verify MongoDB credentials
4. For cloud MongoDB, check IP whitelist

### Issue: M-Pesa Integration Not Working

**Error:** M-Pesa API errors

**Solutions:**
1. Verify M-Pesa credentials in `.env`
2. Check webhook URLs are accessible (use ngrok for local testing)
3. Verify M-Pesa sandbox/production environment matches
4. Check M-Pesa API status

### Issue: TypeScript Compilation Errors

**Error:** Build fails

**Solutions:**
1. Run `npm install` to ensure all dependencies are installed
2. Check TypeScript version compatibility
3. Clear `dist/` folder and rebuild: `rm -rf dist && npm run build`
4. Check for syntax errors in source files

### Issue: Redis Connection Failed

**Error:** Redis connection errors

**Solutions:**
1. Redis is optional - the app will work without it (with reduced functionality)
2. If using Redis, verify `REDIS_URL` in `.env`
3. Ensure Redis server is running: `redis-server`
4. For cloud Redis, verify connection string

## Testing Checklist

- [ ] Server starts without errors
- [ ] Health endpoint returns success
- [ ] Database connection established
- [ ] Can register a new user
- [ ] Can verify email/phone
- [ ] Can login and get JWT token
- [ ] Can access protected endpoints with token
- [ ] Can get user balance
- [ ] Can send tokens (if wallet funded)
- [ ] Admin endpoints work (if admin user exists)
- [ ] M-Pesa endpoints respond (may need actual credentials)
- [ ] Transaction history works
- [ ] Error handling works correctly

## Next Steps

1. **Set up MongoDB**: Use MongoDB Atlas (free tier) or local MongoDB
2. **Set up Redis**: Use Redis Cloud (free tier) or local Redis
3. **Get API Keys**: 
   - ThirdWeb: https://thirdweb.com
   - M-Pesa: https://developer.safaricom.co.ke
   - Africa's Talking: https://africastalking.com
4. **Test Authentication Flow**: Complete user registration
5. **Test Crypto Operations**: Fund wallet and test transfers
6. **Test M-Pesa Integration**: Use sandbox credentials first

## Additional Resources

- **API Documentation**: `API_DOCUMENTATION.md`
- **Frontend Endpoints**: `API_ENDPOINTS_FRONTEND.md`
- **Architecture Overview**: `NEXUSPAY_ARCHITECTURE_OVERVIEW.md`
- **Admin Testing Guide**: `Admin_API_Testing_Guide.md`

## Support

For issues or questions:
1. Check the documentation files in the backend directory
2. Review error logs in console output
3. Check MongoDB/Redis connection status
4. Verify all environment variables are set correctly

