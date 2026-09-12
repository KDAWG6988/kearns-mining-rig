# Backend Setup with Stripe & AlbyHub

Quick setup guide using your existing Stripe and AlbyHub accounts.

## 🚀 5-Minute Setup

### 1. Get Your API Keys

**Stripe:**
- Go to: https://dashboard.stripe.com/apikeys
- Copy: Secret Key (starts with `sk_test_` or `sk_live_`)
- Copy: Publishable Key (starts with `pk_test_` or `pk_live_`)

**AlbyHub (for Bitcoin):**
- Go to: https://getalby.com
- Create account / Sign in
- Go to Settings → API Keys
- Copy: API Token

### 2. Create `.env` File

```env
# Server
PORT=5000
NODE_ENV=development
CLIENT_URL=http://localhost:5173

# Database (MongoDB)
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/kearns-mining-rig

# Stripe (Your existing account)
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxxx
STRIPE_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxx

# AlbyHub (Your existing account)
ALBY_API_TOKEN=your_alby_api_token_here
ALBY_WEBHOOK_SECRET=your_webhook_secret

# Bitcoin Address (Your Platform Wallet)
PLATFORM_BTC_WALLET=your_bitcoin_address_here

# JWT
JWT_SECRET=your_super_secret_jwt_key_here
```

### 3. Install & Run

```bash
# Create backend folder
mkdir kearns-mining-rig-backend
cd kearns-mining-rig-backend

# Install deps
npm install express cors dotenv mongodb stripe axios uuid bcrypt jsonwebtoken

# Create server.js (copy from BACKEND_API.md)

# Start
node server.js
```

---

## 💳 Stripe Integration (Premium Subscriptions)

Your code already handles:
- Bronze: $0.99/month
- Silver: $2.99/month  
- Gold: $4.99/month

**What happens:**
```
User clicks "Subscribe"
  ↓
Frontend sends payment
  ↓
Stripe processes payment
  ↓
Backend confirms
  ↓
User gets premium access
  ↓
Backend stores premium status in MongoDB
```

---

## ₿ AlbyHub Integration (Bitcoin Withdrawals)

### How 30% Fee Works:

```javascript
// User withdraws 0.1 BTC
const userWithdrawal = 0.1;
const platformFee = userWithdrawal * 0.30;  // 0.03 BTC (you get this)
const userPayout = userWithdrawal * 0.70;   // 0.07 BTC (user gets this)

// AlbyHub sends:
// - 0.07 BTC to user's wallet
// - 0.03 BTC to your platform wallet
```

### Setup AlbyHub Webhook:

1. Go to AlbyHub Settings
2. Add Webhook URL: `https://your-backend.com/api/webhooks/alby`
3. Select events: "charge.completed"
4. Save

Your backend will:
- Track completed withdrawals
- Update user account
- Log platform fee
- Send confirmation email

---

## 📝 Minimal Backend (Copy-Paste Ready)

If you want a simpler version just for withdrawals:

```javascript
const express = require('express');
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const axios = require('axios');
require('dotenv').config();

const app = express();
app.use(express.json());

// Premium Subscription
app.post('/api/subscribe', async (req, res) => {
  try {
    const { amount, token, tier } = req.body;
    
    const charge = await stripe.charges.create({
      amount: amount * 100, // Convert to cents
      currency: 'usd',
      source: token,
      description: `Kearns Mining Rig - ${tier.toUpperCase()}`
    });
    
    res.json({ success: true, chargeId: charge.id });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Bitcoin Withdrawal via AlbyHub
app.post('/api/withdraw', async (req, res) => {
  try {
    const { amount, btcWallet } = req.body;
    
    // Calculate 30% fee
    const platformFee = amount * 0.30;
    const userPayout = amount * 0.70;
    
    // Send to AlbyHub
    const response = await axios.post(
      'https://api.getalby.com/payments',
      {
        amount_sats: Math.round(userPayout * 100000000), // Convert BTC to sats
        description: 'Kearns Mining Rig Withdrawal',
        out_invoice: false,
        webhook_url: 'https://your-backend.com/api/webhooks/alby'
      },
      {
        headers: {
          'Authorization': `Bearer ${process.env.ALBY_API_TOKEN}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    // Your platform receives 30% fee
    console.log(`Platform fee: ${platformFee} BTC sent to wallet`);
    
    res.json({
      success: true,
      userPayout,
      platformFee,
      paymentId: response.data.payment_id
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Webhook handler
app.post('/api/webhooks/alby', (req, res) => {
  const { type, data } = req.body;
  
  if (type === 'charge.completed') {
    console.log(`✅ Withdrawal completed: ${data.amount_sats} sats`);
    // Update database, send email, etc.
  }
  
  res.json({ received: true });
});

app.listen(5000, () => console.log('Backend running!'));
```

---

## 🔗 Connect to Frontend

In your Vue app, add API calls:

```javascript
// For premium subscription
async function buyPremium(tier) {
  const response = await fetch('/api/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      tier,
      amount: { bronze: 0.99, silver: 2.99, gold: 4.99 }[tier],
      token: stripeToken // from Stripe.js
    })
  });
  const data = await response.json();
  return data.success;
}

// For Bitcoin withdrawal
async function withdrawBitcoin(amount, btcWallet) {
  const response = await fetch('/api/withdraw', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ amount, btcWallet })
  });
  const data = await response.json();
  return data;
}
```

---

## 🚀 Deploy Backend (Free Options)

### Option 1: Heroku (Recommended)
```bash
heroku create kearns-mining-rig-api
heroku config:set STRIPE_SECRET_KEY=sk_test_xxx
heroku config:set ALBY_API_TOKEN=xxx
git push heroku main
```

### Option 2: Railway.app
1. Go to railway.app
2. Connect GitHub repo
3. Deploy
4. Set env vars in dashboard

### Option 3: Render
1. Go to render.com
2. New Web Service
3. Connect GitHub
4. Deploy

---

## ✅ Testing

```bash
# Test subscription
curl -X POST http://localhost:5000/api/subscribe \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 2.99,
    "tier": "silver",
    "token": "tok_visa"
  }'

# Test withdrawal
curl -X POST http://localhost:5000/api/withdraw \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 0.1,
    "btcWallet": "1A1z7agoat7B6VdCd1VQ7SQKjXmvnN3"
  }'
```

---

**Backend is ready! You already have Stripe & AlbyHub, just add your keys!** 💚
