# Backend API Setup Guide

Complete Node.js/Express backend for Kearns Mining Rig with payment processing.

## 📋 Quick Start (5 minutes)

```bash
# Clone/create backend folder
mkdir kearns-mining-rig-backend
cd kearns-mining-rig-backend

# Initialize
npm init -y

# Install dependencies
npm install express cors dotenv mongodb stripe axios uuid bcrypt jsonwebtoken

# Create .env file (see below)

# Start server
node server.js
```

## 🔧 Environment Setup

Create `.env` file in root:

```env
# Server
PORT=5000
NODE_ENV=development
CLIENT_URL=http://localhost:5173

# Database
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/kearns-mining-rig

# Payments
STRIPE_SECRET_KEY=sk_test_your_key_here
STRIPE_PUBLISHABLE_KEY=pk_test_your_key_here

# Crypto
COINBASE_API_KEY=your_coinbase_key_here
COINBASE_WEBHOOK_SECRET=your_webhook_secret

# JWT
JWT_SECRET=your_super_secret_jwt_key_change_this

# Bitcoin Wallet (Your Platform Wallet)
PLATFORM_BTC_WALLET=your_bitcoin_address_here
```

## 📦 Install Dependencies

```bash
npm install express cors dotenv mongodb stripe axios uuid bcrypt jsonwebtoken
npm install --save-dev nodemon  # For development
```

Update `package.json` scripts:

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

---

## 🚀 Create Server (server.js)

```javascript
const express = require('express');
const cors = require('cors');
const dotenv = require('dotenv');
const { MongoClient, ObjectId } = require('mongodb');
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const axios = require('axios');
const { v4: uuidv4 } = require('uuid');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

dotenv.config();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(express.json());

// MongoDB Connection
let db;
const client = new MongoClient(process.env.MONGODB_URI);

async function connectDB() {
  try {
    await client.connect();
    db = client.db('kearns-mining-rig');
    console.log('✅ MongoDB connected');
  } catch (error) {
    console.error('❌ MongoDB error:', error);
    process.exit(1);
  }
}

// Auth Middleware
function verifyToken(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = decoded.userId;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// ============ USER ROUTES ============

// Register
app.post('/api/auth/register', async (req, res) => {
  try {
    const { email, password } = req.body;
    const users = db.collection('users');
    
    const existing = await users.findOne({ email });
    if (existing) return res.status(400).json({ error: 'User exists' });
    
    const hashedPassword = await bcrypt.hash(password, 10);
    const result = await users.insertOne({
      email,
      password: hashedPassword,
      createdAt: new Date(),
      ore: 0,
      bankBalance: 0,
      totalEarnings: 0,
      monthlyEarnings: 0,
      referralEarnings: 0,
      referralCount: 0,
      isPremium: false,
      premiumTier: null,
      withdrawalHistory: []
    });
    
    const token = jwt.sign({ userId: result.insertedId }, process.env.JWT_SECRET);
    res.json({ success: true, token, userId: result.insertedId });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Login
app.post('/api/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    const users = db.collection('users');
    
    const user = await users.findOne({ email });
    if (!user) return res.status(400).json({ error: 'User not found' });
    
    const validPassword = await bcrypt.compare(password, user.password);
    if (!validPassword) return res.status(400).json({ error: 'Invalid password' });
    
    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET);
    res.json({ success: true, token, userId: user._id });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get User Data
app.get('/api/user/profile', verifyToken, async (req, res) => {
  try {
    const users = db.collection('users');
    const user = await users.findOne({ _id: new ObjectId(req.userId) });
    
    if (!user) return res.status(404).json({ error: 'User not found' });
    
    res.json({
      email: user.email,
      ore: user.ore,
      bankBalance: user.bankBalance,
      totalEarnings: user.totalEarnings,
      monthlyEarnings: user.monthlyEarnings,
      referralEarnings: user.referralEarnings,
      referralCount: user.referralCount,
      isPremium: user.isPremium,
      premiumTier: user.premiumTier,
      withdrawalHistory: user.withdrawalHistory || []
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Update Game Progress
app.post('/api/user/progress', verifyToken, async (req, res) => {
  try {
    const { ore, bankBalance, upgrades, monthlyEarnings } = req.body;
    const users = db.collection('users');
    
    await users.updateOne(
      { _id: new ObjectId(req.userId) },
      { $set: { ore, bankBalance, upgrades, monthlyEarnings, updatedAt: new Date() } }
    );
    
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============ PREMIUM ROUTES ============

// Create Premium Subscription
app.post('/api/premium/subscribe', verifyToken, async (req, res) => {
  try {
    const { tier, paymentMethodId } = req.body;
    const prices = { bronze: 99, silver: 299, gold: 499 }; // in cents
    
    const paymentIntent = await stripe.paymentIntents.create({
      amount: prices[tier],
      currency: 'usd',
      payment_method: paymentMethodId,
      confirm: true,
      metadata: { userId: req.userId, tier }
    });
    
    if (paymentIntent.status === 'succeeded') {
      const users = db.collection('users');
      await users.updateOne(
        { _id: new ObjectId(req.userId) },
        { 
          $set: { 
            isPremium: true, 
            premiumTier: tier,
            subscriptionEndsAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000) // 30 days
          } 
        }
      );
      
      res.json({ success: true, message: 'Premium activated!' });
    } else {
      res.status(400).json({ error: 'Payment failed' });
    }
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============ WITHDRAWAL ROUTES ============

// Process Bitcoin Withdrawal (with 30% fee)
app.post('/api/withdraw', verifyToken, async (req, res) => {
  try {
    const { amount, btcWallet } = req.body;
    const users = db.collection('users');
    const withdrawals = db.collection('withdrawals');
    
    const user = await users.findOne({ _id: new ObjectId(req.userId) });
    if (!user || user.bankBalance < amount) {
      return res.status(400).json({ error: 'Insufficient balance' });
    }
    
    // Calculate 30% fee
    const platformFee = amount * 0.30;
    const userPayout = amount * 0.70;
    
    // Create withdrawal record
    const withdrawalId = uuidv4();
    await withdrawals.insertOne({
      withdrawalId,
      userId: req.userId,
      amount,
      userPayout,
      platformFee,
      btcWallet,
      status: 'pending',
      createdAt: new Date(),
      processedAt: null
    });
    
    // Deduct from user's bank balance
    await users.updateOne(
      { _id: new ObjectId(req.userId) },
      { $set: { bankBalance: user.bankBalance - amount } }
    );
    
    res.json({ 
      success: true, 
      withdrawalId,
      message: `Withdrawal pending. You'll receive ${userPayout} BTC. Platform fee: ${platformFee} BTC`,
      userPayout,
      platformFee
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Process Withdrawal via Coinbase Commerce
app.post('/api/withdraw/coinbase', verifyToken, async (req, res) => {
  try {
    const { amount, btcWallet } = req.body;
    const users = db.collection('users');
    
    const user = await users.findOne({ _id: new ObjectId(req.userId) });
    if (!user || user.bankBalance < amount) {
      return res.status(400).json({ error: 'Insufficient balance' });
    }
    
    const platformFee = amount * 0.30;
    const userPayout = amount * 0.70;
    
    // Call Coinbase Commerce API
    const coinbaseResponse = await axios.post(
      'https://api.commerce.coinbase.com/charges',
      {
        name: 'Kearns Mining Rig Withdrawal',
        description: `Withdrawal of ${userPayout} BTC`,
        local_price: {
          amount: userPayout,
          currency: 'BTC'
        },
        pricing_type: 'fixed_price',
        metadata: {
          userId: req.userId,
          withdrawalAmount: userPayout,
          platformFee: platformFee,
          destination: btcWallet
        }
      },
      {
        headers: {
          'X-CC-Api-Key': process.env.COINBASE_API_KEY,
          'X-CC-Version': '2018-03-22'
        }
      }
    );
    
    // Store withdrawal record
    const withdrawals = db.collection('withdrawals');
    await withdrawals.insertOne({
      chargeId: coinbaseResponse.data.data.id,
      userId: req.userId,
      amount,
      userPayout,
      platformFee,
      btcWallet,
      status: 'pending_payment',
      createdAt: new Date()
    });
    
    res.json({
      success: true,
      chargeId: coinbaseResponse.data.data.id,
      paymentUrl: coinbaseResponse.data.data.hosted_url,
      userPayout,
      platformFee
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get Withdrawal History
app.get('/api/withdrawals/history', verifyToken, async (req, res) => {
  try {
    const withdrawals = db.collection('withdrawals');
    const history = await withdrawals
      .find({ userId: req.userId })
      .sort({ createdAt: -1 })
      .limit(20)
      .toArray();
    
    res.json(history);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============ REFERRAL ROUTES ============

// Get Referral Link
app.get('/api/referral/link', verifyToken, async (req, res) => {
  try {
    const users = db.collection('users');
    const user = await users.findOne({ _id: new ObjectId(req.userId) });
    
    const referralLink = `https://kearns-mining-rig.vercel.app?ref=${req.userId}`;
    res.json({ referralLink, referralCount: user.referralCount || 0 });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Track Referral Sign-up
app.post('/api/referral/track', async (req, res) => {
  try {
    const { referrerId, newUserId } = req.body;
    const users = db.collection('users');
    
    // Add to referrer's count
    await users.updateOne(
      { _id: new ObjectId(referrerId) },
      { $inc: { referralCount: 1, referralEarnings: 5 } } // $5 per referral
    );
    
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============ WEBHOOK ROUTES ============

// Stripe Webhook
app.post('/api/webhooks/stripe', express.raw({ type: 'application/json' }), async (req, res) => {
  const sig = req.headers['stripe-signature'];
  
  try {
    const event = stripe.webhooks.constructEvent(
      req.body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET
    );
    
    if (event.type === 'payment_intent.succeeded') {
      console.log('✅ Payment succeeded:', event.data.object);
    }
    
    res.json({ received: true });
  } catch (error) {
    res.status(400).send(`Webhook Error: ${error.message}`);
  }
});

// Coinbase Webhook
app.post('/api/webhooks/coinbase', async (req, res) => {
  try {
    const { data } = req.body;
    const withdrawals = db.collection('withdrawals');
    
    if (data.status === 'COMPLETED') {
      // Mark withdrawal as complete
      await withdrawals.updateOne(
        { chargeId: data.id },
        { $set: { status: 'completed', processedAt: new Date() } }
      );
      
      // Log platform fee to your wallet
      console.log(`✅ Withdrawal completed. Platform fee received.`);
    }
    
    res.json({ received: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============ ADMIN ROUTES ============

// Get Platform Earnings
app.get('/api/admin/earnings', verifyToken, async (req, res) => {
  try {
    // Verify admin (you would add proper admin check)
    const withdrawals = db.collection('withdrawals');
    
    const earnings = await withdrawals.aggregate([
      { $match: { status: 'completed' } },
      { $group: { _id: null, totalFees: { $sum: '$platformFee' } } }
    ]).toArray();
    
    res.json({
      totalPlatformFees: earnings[0]?.totalFees || 0,
      currency: 'BTC'
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============ HEALTH CHECK ============

app.get('/api/health', (req, res) => {
  res.json({ status: 'Backend running! ✅' });
});

// Start Server
connectDB().then(() => {
  app.listen(PORT, () => {
    console.log(`🚀 Server running on port ${PORT}`);
    console.log(`📝 API: http://localhost:${PORT}`);
  });
});
```

---

## 🗄️ MongoDB Schemas

Create file `schemas.js`:

```javascript
// Users Collection
db.users.createIndex({ email: 1 }, { unique: true });

// Users Sample Structure
{
  "_id": ObjectId,
  "email": "user@example.com",
  "password": "hashed_password",
  "ore": 1000,
  "bankBalance": 5000,
  "totalEarnings": 0.5,
  "monthlyEarnings": 150,
  "referralEarnings": 25,
  "referralCount": 3,
  "isPremium": true,
  "premiumTier": "silver",
  "subscriptionEndsAt": ISODate,
  "withdrawalHistory": [
    {
      "id": "uuid",
      "amount": 0.1,
      "userPayout": 0.07,
      "platformFee": 0.03,
      "date": ISODate,
      "status": "completed"
    }
  ],
  "createdAt": ISODate,
  "updatedAt": ISODate
}

// Withdrawals Collection
{
  "_id": ObjectId,
  "chargeId": "string",
  "userId": ObjectId,
  "amount": 0.1,
  "userPayout": 0.07,
  "platformFee": 0.03,
  "btcWallet": "1A1z7agoat...",
  "status": "completed|pending",
  "createdAt": ISODate,
  "processedAt": ISODate
}

// Transactions Collection
{
  "_id": ObjectId,
  "userId": ObjectId,
  "type": "purchase|withdrawal|referral",
  "amount": 5.00,
  "description": "Premium Silver",
  "createdAt": ISODate
}
```

---

## 🚀 Deploy to Heroku (FREE)

1. **Install Heroku CLI**
```bash
npm install -g heroku
heroku login
```

2. **Create Heroku app**
```bash
heroku create kearns-mining-rig-api
```

3. **Set environment variables**
```bash
heroku config:set MONGODB_URI=your_mongodb_uri
heroku config:set STRIPE_SECRET_KEY=your_stripe_key
heroku config:set COINBASE_API_KEY=your_coinbase_key
heroku config:set JWT_SECRET=your_jwt_secret
```

4. **Deploy**
```bash
git push heroku main
heroku logs --tail
```

---

## 🔗 Connect Frontend to Backend

Update `src/main.js`:

```javascript
// API Configuration
const API_URL = process.env.VUE_APP_API_URL || 'http://localhost:5000/api'

// Export for use in components
export const api = {
  async request(method, endpoint, data = null) {
    const token = localStorage.getItem('authToken')
    const options = {
      method,
      headers: {
        'Content-Type': 'application/json',
        ...(token && { Authorization: `Bearer ${token}` })
      }
    }
    if (data) options.body = JSON.stringify(data)
    
    const response = await fetch(`${API_URL}${endpoint}`, options)
    return response.json()
  }
}
```

---

## ✅ Testing Endpoints

```bash
# Health check
curl http://localhost:5000/api/health

# Register
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Withdraw
curl -X POST http://localhost:5000/api/withdraw \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"amount":0.1,"btcWallet":"1A1z7agoat..."}'
```

---

**Backend is ready! Next: Android wrapper setup.** 🚀
