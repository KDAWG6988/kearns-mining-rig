# Google Play Store Setup Guide

## ⚠️ Important: Web App vs Native App

Your current app is a **web app** (runs in browser). To publish on Google Play Store, you need:

### Option A: Wrap Web App as Native (Easier, Recommended)

Use **Capacitor** or **Apache Cordova** to wrap your Vue web app into an Android app.

#### Step 1: Install Capacitor
```bash
npm install @capacitor/core @capacitor/cli
npx cap init
```

#### Step 2: Add Android Platform
```bash
npx cap add android
```

#### Step 3: Build Web App
```bash
npm run build
```

#### Step 4: Sync to Android
```bash
npx cap sync
```

#### Step 5: Open in Android Studio
```bash
npx cap open android
```

#### Step 6: Build APK in Android Studio
- Tools → Build → Build Bundle(s) / APK(s)
- Select "Build APK"

### Option B: Rewrite as Native Android App (Advanced)

Build in **Kotlin/Java** using Android Studio directly. (Much more work)

---

## 📝 Google Play Store Submission Checklist

### 1. Developer Account Setup
- [ ] Create Google Play Developer account ($25 one-time fee)
- [ ] Add payment method
- [ ] Complete developer profile

### 2. App Configuration
- [ ] App name: "Kearns Mining Rig"
- [ ] Short description: "Tap to mine ore and earn real Bitcoin"
- [ ] Full description: Include monetization transparency
- [ ] App icon (512x512 PNG)
- [ ] Screenshots (5 minimum, showing gameplay)
- [ ] Feature graphic (1024x500)
- [ ] Video demo (optional but recommended)

### 3. Content Rating
- [ ] Fill out content questionnaire
- [ ] Expected rating: PEGI 3 or equivalent

### 4. Monetization & Ads
- [ ] Mark as free with in-app purchases
- [ ] Declare AdSense usage
- [ ] Add AdSense publisher ID
- [ ] Add privacy policy URL

### 5. Privacy & Security
- [ ] Upload privacy policy
- [ ] Declare target audience (13+)
- [ ] Declare permissions needed:
  - [ ] Internet
  - [ ] Location (optional, for regional ads)

### 6. Testing
- [ ] Test on Android emulator
- [ ] Test all IAP flows
- [ ] Test ad displays
- [ ] Test withdrawal flow
- [ ] Test offline functionality

### 7. Upload to Play Store
- [ ] Generate signing key
- [ ] Sign APK/Bundle
- [ ] Upload to Play Console
- [ ] Set pricing ($0 free, IAP for monetization)
- [ ] Set regions/countries
- [ ] Submit for review (24-72 hours)

---

## 🔑 Required Files for Submission

```
/app
├── app-release.aab          (Android App Bundle)
├── privacy-policy.html      (Required!)
├── app-icon-512x512.png
├── screenshots/
│   ├── screenshot1.png
│   ├── screenshot2.png
│   └── ... (5+ total)
└── feature-graphic.png
```

---

## 💰 Monetization Policy

Google Play requires **clear transparency**:

### In Your App Store Listing:
```
"KEARNS MINING RIG - Play the Tap-to-Mine Game

🎮 Features:
• Free gameplay with optional premium features
• Premium subscriptions ($0.99-$4.99/month)
• In-app purchases for cosmetics and perks
• Ad-supported free experience
• Real Bitcoin withdrawals available

💬 IMPORTANT: Earnings are not guaranteed. Actual Bitcoin payouts depend on:
- Your engagement level
- Minimum withdrawal threshold (0.001 BTC)
- Current Bitcoin exchange rates
- Successful payment processing"
```

### Privacy Policy Must Include:
- Data collection practices
- Bitcoin wallet security
- Third-party integrations (Stripe, Firebase, etc.)
- User rights

---

## 🚀 Submission Steps

### 1. Create Bundle for Play Store
```bash
# In Android Studio
Build → Build Bundle(s) / APK(s) → Build App Bundle
```

### 2. Sign the Bundle
- Android Studio will prompt for signing key
- Create new key or use existing
- Save keystore file safely!

### 3. Upload to Google Play Console
1. Go to https://play.google.com/console
2. Click "Kearns Mining Rig"
3. Navigate to "Release" → "Production"
4. Click "Create new release"
5. Upload signed bundle
6. Add release notes
7. Review content rating
8. Submit for review

### 4. Wait for Review
- Google typically reviews in 24-72 hours
- May request changes to monetization disclosure
- Once approved, app goes live!

---

## 🔴 Common Rejection Reasons (Avoid These!)

❌ **Misleading earnings claims** - Don't promise specific Bitcoin amounts
❌ **Gambling references** - Don't call it a "lottery" or "gambling game"
❌ **Fake endorsements** - Don't claim to be endorsed by crypto companies
❌ **Poor monetization clarity** - Must clearly disclose all paywalls
❌ **Excessive ads** - Don't spam ads to users
❌ **Unsafe payment handling** - Must encrypt wallets and payments

✅ **What Gets Approved:**
- Clear "free with optional in-app purchases"
- Transparent earnings disclaimers
- Professional UI/UX
- Working withdrawal system
- Good user reviews

---

## 💡 Tips for Success

1. **Start with Beta Testing**
   - Use internal testing track first
   - Get 20+ testers to review
   - Gather feedback before public release

2. **Optimize Listing for Discoverability**
   - Keywords: "mining game", "idle game", "tap game", "bitcoin"
   - Engaging screenshots showing earning mechanics
   - High-quality app icon

3. **Launch Strategy**
   - Start in tier-1 countries (US, UK, Canada)
   - Monitor reviews closely first week
   - Fix bugs reported by users quickly
   - Update frequently (shows active development)

4. **Monetization Success**
   - Premium tier conversion rate target: 2-5%
   - Ad revenue: ~$5-15 per 1K impressions
   - Average user lifetime value: $2-10

---

## 📊 Expected Timeline

| Phase | Timeline |
|-------|----------|
| App Wrapping | 2-4 hours |
| Testing | 4-8 hours |
| Store Listing Setup | 2-4 hours |
| First Submission | 24-72 hours |
| Total to Launch | 3-5 days |

---

## 🔧 Backend Requirements (To Enable Real Payments)

Before publishing, ensure these are integrated:

### For Premium Subscriptions:
- [ ] Stripe/Paddle API integration
- [ ] Receipt verification
- [ ] Subscription management

### For Bitcoin Withdrawals:
- [ ] Coinbase Commerce API
- [ ] Wallet validation
- [ ] Payment processing (24-48 hour queue)

### For User Accounts:
- [ ] Firebase Authentication
- [ ] User database (Firestore/MongoDB)
- [ ] Withdrawal history tracking

### For Ad Revenue:
- [ ] Google AdSense setup
- [ ] Ad placement in free tier
- [ ] Revenue tracking

---

## 📞 Support Resources

- **Google Play Console Help**: https://support.google.com/googleplay/
- **Capacitor Docs**: https://capacitorjs.com/docs/
- **Android Studio Guide**: https://developer.android.com/studio

---

**Ready to launch? Start with the Capacitor setup above! 🚀**
