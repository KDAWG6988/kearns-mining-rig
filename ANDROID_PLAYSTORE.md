# Android Wrapper & Play Store Setup Guide

Complete step-by-step guide to wrap your Vue app as Android and publish to Google Play Store.

## 🎯 Overview

Your web app → Capacitor wraps it → Android APK/Bundle → Google Play Store

**Total time:** 4-6 hours (mostly following steps)

---

## PHASE 1: LOCAL SETUP (30 minutes)

### Step 1: Install Required Tools

```bash
# Install Java JDK 11+
# Download from: https://www.oracle.com/java/technologies/downloads/

# Install Android Studio
# Download from: https://developer.android.com/studio

# After install, set ANDROID_HOME environment variable
# On Mac: echo 'export ANDROID_HOME=/Users/YOUR_USERNAME/Library/Android/sdk' >> ~/.zshrc
# On Windows: Set ANDROID_HOME=C:\Users\YOUR_USERNAME\AppData\Local\Android\sdk
```

### Step 2: Add Capacitor to Your Project

```bash
cd kearns-mining-rig  # Your Vue app folder

# Install Capacitor
npm install @capacitor/core @capacitor/cli
npm install @capacitor/android

# Initialize Capacitor
npx cap init "Kearns Mining Rig" "com.kdawg6988.kearnsminingen" --web-dir=dist

# This creates capacitor.config.json with:
# - appId: com.kdawg6988.kearnsminingen
# - appName: Kearns Mining Rig
# - webDir: dist
```

### Step 3: Build Vue App for Production

```bash
# Build optimized production version
npm run build

# Output goes to: dist/
# This is ~2-3MB after compression
```

### Step 4: Add Android Platform

```bash
# Add Android to Capacitor
npx cap add android

# This creates android/ folder with Android project
```

### Step 5: Sync Web Assets to Android

```bash
# Copy dist files to Android project
npx cap sync android

# This keeps Android app in sync with your web code
```

---

## PHASE 2: ANDROID STUDIO BUILD (1-2 hours)

### Step 6: Open in Android Studio

```bash
# Open Android project
npx cap open android

# This opens Android Studio automatically
```

### Step 7: Configure App Details in Android Studio

In Android Studio, navigate to:
```
app/manifests/AndroidManifest.xml
```

Verify these settings:
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.kdawg6988.kearnsminingen">
    
    <!-- Add required permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    
    <application
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher">
        
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:theme="@style/AppTheme"
            android:launchMode="singleTask"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale"
            android:windowSoftInputMode="adjustResize">
            
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

### Step 8: Update App Icon

Replace default icon with your own:

```
app/src/main/res/mipmap-*/ic_launcher.png
```

Sizes needed:
- mdpi: 48x48
- hdpi: 72x72
- xhdpi: 96x96
- xxhdpi: 144x144
- xxxhdpi: 192x192

### Step 9: Test on Emulator

In Android Studio:
1. Click **AVD Manager** (top right)
2. Create virtual device (e.g., Pixel 5, Android 12)
3. Click **Run** button (or Shift+F10)
4. Select emulator
5. App launches in emulator
6. **Test everything:** mining, taps, withdrawals, premium

Verify:
- ✅ Game mechanics work
- ✅ Bank deposit/withdraw works
- ✅ Premium features appear
- ✅ No console errors
- ✅ UI is responsive

---

## PHASE 3: CREATE SIGNING KEY (30 minutes)

### Step 10: Generate Release Key

In Android Studio:
1. Go to **Build** → **Generate Signed Bundle / APK**
2. Choose **Android App Bundle** (AAB) - recommended for Play Store
3. Click **Create new** under "Key store path"

Fill in:
```
Key store path: /path/to/kearns-mining.jks  (save this!)
Password: YourStrongPassword123!
Key alias: kearns-mining-key
Key password: YourStrongPassword123!
Common name: Your Name
Organization Unit: Your Company
Organization: Your Company
Country: US
```

**⚠️ IMPORTANT: Save the keystore file! You'll need it for future updates.**

```bash
# Never commit this to GitHub!
echo "*.jks" >> .gitignore
git add .gitignore && git commit -m "Ignore keystore"
```

### Step 11: Build Release Bundle

Android Studio automatically:
1. Compiles your app
2. Signs with your key
3. Creates `app-release.aab` (~10-15MB)

Location: `android/app/release/app-release.aab`

**Congratulations! You now have a signed Android app! 🎉**

---

## PHASE 4: GOOGLE PLAY STORE SETUP (1-2 hours)

### Step 12: Create Google Play Developer Account

1. Go to: https://play.google.com/console
2. Click **Sign up or sign in**
3. Pay $25 one-time developer fee
4. Add payment method
5. Complete developer profile:
   - Name
   - Email
   - Website (optional)
   - Phone

### Step 13: Create New App

1. In Play Console, click **Create app**
2. Fill in:
   - **App name:** Kearns Mining Rig
   - **Default language:** English
   - **App or game:** Game
   - **Free or paid:** Free
   - Click **Create app**

### Step 14: Set Up App Content

In Play Console, go to **Policy** section:

#### Target Audience
- Select: **Age 13 and older**

#### Content Rating
1. Click **Content rating**
2. Fill questionnaire (5 minutes)
3. Get rating (usually PEGI 3)

#### Privacy Policy
Must create privacy policy at: https://privacypolicygenerator.info/

Include:
```
1. Data We Collect:
   - Email (for authentication)
   - Game progress (stored locally)
   - Bitcoin wallet address (encrypted)

2. How We Use Data:
   - Authentication only
   - Game save data
   - Withdrawal processing

3. Third-party Services:
   - Stripe (payments)
   - AlbyHub (Bitcoin)
   - Google Analytics (optional)

4. User Rights:
   - Can delete account
   - Can request data deletion
   - No data sold to third parties

5. 30% Withdrawal Fee:
   - Platform charges 30% on all Bitcoin withdrawals
   - Fee covers payment processing, security, maintenance
```

Upload privacy policy URL in Play Console.

#### Monetization
1. Click **Monetization setup**
2. Select: **Free with in-app purchases**
3. Add in-app products:
   - Bronze Premium: $0.99
   - Silver Premium: $2.99
   - Gold Premium: $4.99
   - Golden Pickaxe cosmetic: $2.99
   - Ad-Free Pass: $0.99
   - Battle Pass: $4.99

---

## PHASE 5: STORE LISTING (1-2 hours)

### Step 15: Create Store Assets

You need:

**1. App Icon (512x512 PNG)**
- High quality
- No text
- Safe zone: inner 64x64
- Download template: https://developer.android.com/develop/ui/views/appwidgets/app-widget-host#RecipientAppWidget

**2. Screenshots (5-8 minimum)**
Size: 1080x1920 pixels (9:16 aspect ratio)

Screenshots to include:
- Screenshot 1: Game home screen with "Tap to Mine"
- Screenshot 2: Mining action, showing ore accumulation
- Screenshot 3: Bank deposit/withdraw system
- Screenshot 4: Premium tier selection
- Screenshot 5: Account dashboard showing earnings
- Screenshot 6: Withdrawal screen with 30% fee clearly shown
- Screenshot 7: Referral system

**3. Feature Graphic (1024x500)**
- Your app banner
- Shows game title + key features
- Professional design

**4. Short Description (80 chars max)**
```
⛏️ Tap to mine ore and earn real Bitcoin rewards! 💰
```

**5. Full Description (4000 chars)**
```
🎮 KEARNS MINING RIG - Tap to Earn Bitcoin!

⛏️ GAMEPLAY:
- Tap the mine button to collect ore
- Build an automatic mining empire
- Level up your mining operation
- Collect rewards and withdrawals

💰 EARN REAL BITCOIN:
- Accumulate ore in the game
- Convert to your bank
- Withdraw to real Bitcoin wallet
- Minimum withdrawal: 0.001 BTC (~$25)

🏆 PREMIUM MEMBERSHIP:
- Bronze: $0.99/month - 50% mining boost
- Silver: $2.99/month - 100% boost + $5/week bonus
- Gold: $4.99/month - 200% boost + $15/week bonus

⭐ FEATURES:
✅ Free to play (premium optional)
✅ Real Bitcoin withdrawals
✅ No ads (or premium to remove)
✅ Save progress automatically
✅ Earn through referrals
✅ In-app cosmetics

⚠️ IMPORTANT - READ BEFORE PLAYING:
- Bitcoin withdrawals subject to 30% platform fee
- Earnings are NOT guaranteed - depends on engagement
- Minimum payout: 0.001 BTC
- Processing time: 24-48 hours
- Premium features entirely optional

🚀 Start mining today and build your Bitcoin fortune!
```

**6. Video (Optional but recommended)**
30-second gameplay video showing:
- Tapping to mine
- Ore accumulation
- Bank system
- Withdrawal flow

### Step 16: Upload Assets to Play Console

1. Go to **Store presence** → **Main store listing**
2. Upload app icon
3. Upload feature graphic
4. Upload 5+ screenshots
5. Add short description
6. Add full description
7. Add support email
8. Add privacy policy URL
9. Add website (optional)

---

## PHASE 6: SUBMISSION (30 minutes)

### Step 17: Upload App Bundle

1. In Play Console, go to **Release** → **Production**
2. Click **Create new release**
3. Upload `app-release.aab` file
4. Add release notes:
```
🚀 Version 1.0 Launch

Initial release of Kearns Mining Rig!

Features:
✅ Tap to mine gameplay
✅ Real Bitcoin withdrawals (30% platform fee)
✅ Premium subscriptions available
✅ In-app purchases
✅ Bank system
✅ Referral rewards

Thank you for playing!
```

### Step 18: Review & Submit

1. Android Studio reviews app:
   - Targets API 31+? ✅
   - 64-bit support? ✅
   - Content rating selected? ✅
   - Privacy policy set? ✅

2. Click **Review** to check compliance
3. Click **Send for review**
4. **App submitted!** 🎉

---

## ⏳ REVIEW PROCESS (24-72 hours)

Google reviews your app for:
- ✅ Functionality
- ✅ Compliance with policies
- ✅ Monetization transparency
- ✅ Security
- ✅ Performance

You'll get email notification when:
- ✅ Approved → App goes live on Play Store
- ⚠️ Rejected → You'll get specific reasons (usually quick fixes)

---

## 📊 COMMON REJECTION REASONS (AND HOW TO AVOID)

❌ **Hidden fees** 
- ✅ FIX: Show 30% fee prominently in withdrawal screen + screenshots

❌ **Gambling language**
- ✅ FIX: Use "game" not "lottery"; use "earn" not "gamble"

❌ **Fake Bitcoin claims**
- ✅ FIX: Show real withdrawal system with minimum threshold

❌ **Poor monetization disclosure**
- ✅ FIX: Screenshot showing withdrawal fee; privacy policy mentioning fee

❌ **Unsafe payment handling**
- ✅ FIX: Use Stripe/AlbyHub; encrypt wallet addresses

❌ **App crashes on startup**
- ✅ FIX: Test on emulator thoroughly

❌ **Target API too low**
- ✅ FIX: Update to API 31+ in `build.gradle`

---

## 🎉 LAUNCH DAY!

Once approved:

1. **App appears on Play Store** (automatic)
2. **Promote on social media:**
   - Twitter: "My tap-to-mine game is live! Earn Bitcoin! 🎮⛏️💰"
   - Reddit: Post in r/AndroidGaming, r/Games, r/Bitcoin
   - Discord: Join gaming/crypto communities

3. **Monitor:**
   - User reviews
   - Crash reports
   - Revenue
   - Downloads

4. **Update frequently:**
   - Fix bugs
   - Add features
   - Improve monetization
   - Each update brings visibility boost

---

## 📊 POST-LAUNCH CHECKLIST

- [ ] Monitor Google Play Console daily first week
- [ ] Respond to user reviews
- [ ] Fix crashes reported
- [ ] Check withdrawal processing
- [ ] Verify Stripe/AlbyHub payouts
- [ ] Collect analytics
- [ ] Plan updates
- [ ] Monitor user retention

---

## 🔧 TROUBLESHOOTING

**App won't launch in emulator?**
```bash
npx cap sync android
npx cap open android
# Click Run again
```

**Assets not updating?**
```bash
npx cap sync android
# Rebuild in Android Studio
```

**Icon shows default?**
```bash
# Make sure PNG is in right folder:
android/app/src/main/res/mipmap-xxxhdpi/ic_launcher.png
# Clean & rebuild
```

**Signature error on upload?**
```bash
# Use same keystore file you created
# Or generate new one if lost (won't be able to update old app)
```

---

## 📞 Resources

- Android Studio docs: https://developer.android.com/studio
- Capacitor docs: https://capacitorjs.com/docs/
- Play Console help: https://support.google.com/googleplay/
- App signing: https://developer.android.com/studio/publish/app-signing
- Play policies: https://play.google.com/about/developer-content-policy/

---

**Ready to launch? Follow the phases above! 🚀**
