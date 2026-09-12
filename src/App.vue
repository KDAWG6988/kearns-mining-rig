<template>
  <div id="app" class="game-container">
    <header class="game-header">
      <div class="header-top">
        <h1>⛏️ Kearns Mining Rig</h1>
        <div class="header-buttons">
          <button @click="showPremium = true" class="premium-btn" v-if="!isPremium">
            ⭐ Go Premium
          </button>
          <button @click="showAccount = true" class="account-btn">
            👤 Account
          </button>
        </div>
      </div>
      <p class="subtitle">Tap to Mine & Earn Real Bitcoin</p>
    </header>

    <!-- PREMIUM MODAL -->
    <div v-if="showPremium" class="modal-overlay" @click="showPremium = false">
      <div class="modal" @click.stop>
        <button class="close-btn" @click="showPremium = false">✕</button>
        <h2>⭐ Premium Membership</h2>
        <div class="premium-tiers">
          <div class="tier">
            <h3>Bronze</h3>
            <p class="price">$0.99/month</p>
            <ul>
              <li>✅ Remove ads</li>
              <li>✅ +50% mining speed</li>
              <li>✅ Daily bonus ore</li>
            </ul>
            <button @click="selectPremium('bronze')" class="buy-btn">Subscribe</button>
          </div>
          <div class="tier featured">
            <h3>Silver ⭐</h3>
            <p class="price">$2.99/month</p>
            <ul>
              <li>✅ All Bronze perks</li>
              <li>✅ +100% mining speed</li>
              <li>✅ Weekly $5 bonus</li>
              <li>✅ Exclusive cosmetics</li>
            </ul>
            <button @click="selectPremium('silver')" class="buy-btn featured-btn">Subscribe</button>
          </div>
          <div class="tier">
            <h3>Gold</h3>
            <p class="price">$4.99/month</p>
            <ul>
              <li>✅ All Silver perks</li>
              <li>✅ +200% mining speed</li>
              <li>✅ Weekly $15 bonus</li>
              <li>✅ VIP status</li>
              <li>✅ 30% referral bonus</li>
            </ul>
            <button @click="selectPremium('gold')" class="buy-btn">Subscribe</button>
          </div>
        </div>
      </div>
    </div>

    <!-- ACCOUNT MODAL -->
    <div v-if="showAccount" class="modal-overlay" @click="showAccount = false">
      <div class="modal" @click.stop>
        <button class="close-btn" @click="showAccount = false">✕</button>
        <h2>💰 Your Account</h2>
        <div class="account-stats">
          <div class="stat-box">
            <span>Lifetime Earnings</span>
            <span class="amount">{{ totalEarnings.toFixed(2) }} BTC</span>
          </div>
          <div class="stat-box">
            <span>Available Balance</span>
            <span class="amount">{{ withdrawBalance.toFixed(2) }} BTC</span>
          </div>
          <div class="stat-box">
            <span>This Month ({{ currentMonth }})</span>
            <span class="amount">${{ monthlyEarnings.toFixed(2) }}</span>
          </div>
          <div class="stat-box">
            <span>Referral Commissions</span>
            <span class="amount">${{ referralEarnings.toFixed(2) }}</span>
          </div>
        </div>

        <div class="withdrawal-section">
          <h3>Withdraw Bitcoin</h3>
          <p v-if="withdrawBalance <= 0" class="warning">Minimum balance: 0.001 BTC</p>
          <input 
            v-else
            v-model.number="withdrawAmount" 
            type="number" 
            placeholder="Amount (BTC)"
            :max="withdrawBalance"
            step="0.001"
          >
          <input v-else v-model="btcWallet" placeholder="Your Bitcoin wallet address">
          <button 
            v-if="withdrawBalance > 0"
            @click="withdrawBitcoin" 
            :disabled="withdrawAmount <= 0 || !btcWallet"
            class="buy-btn"
          >
            Withdraw {{ withdrawAmount }} BTC
          </button>
        </div>

        <div class="referral-section">
          <h3>🔗 Referral Link</h3>
          <p>Earn 20% commission when friends upgrade!</p>
          <div class="referral-link">
            <input v-model="referralLink" disabled>
            <button @click="copyReferral" class="copy-btn">Copy</button>
          </div>
          <p class="referral-count">Total Referrals: {{ referralCount }}</p>
        </div>
      </div>
    </div>

    <main class="game-main">
      <!-- PREMIUM BANNER -->
      <div v-if="isPremium" class="premium-banner">
        ⭐ Premium Active | Mining Boost: {{ premiumBoost }}% | Next Bonus: ${{ nextBonusAmount }}
      </div>

      <!-- AD PLACEHOLDER -->
      <div v-if="!isPremium" class="ad-container">
        <p>[ Advertisement - Google AdSense ]</p>
      </div>

      <div class="stats-panel">
        <div class="stat">
          <span class="stat-label">Ore Mined:</span>
          <span class="stat-value">{{ Math.floor(ore) }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">Bank Balance:</span>
          <span class="stat-value">{{ Math.floor(bankBalance) }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">💰 Value:</span>
          <span class="stat-value">${{ totalOreValue.toFixed(2) }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">Mining Rate:</span>
          <span class="stat-value">{{ miningRate.toFixed(2) }}/s</span>
        </div>
        <div class="stat">
          <span class="stat-label">Level:</span>
          <span class="stat-value">{{ level }}</span>
        </div>
      </div>

      <div class="banking-area">
        <div class="bank-controls">
          <button @click="depositOre" class="bank-btn deposit-btn" :disabled="ore <= 0">
            🏦 Deposit Ore
          </button>
          <button @click="withdrawOre" class="bank-btn withdraw-btn" :disabled="bankBalance <= 0">
            💰 Withdraw Ore
          </button>
        </div>
        <p class="bank-info" v-if="lastTransaction">{{ lastTransaction }}</p>
      </div>

      <div class="mining-area">
        <button @click="mine" class="mine-button">
          💎 TAP TO MINE
        </button>
        <p class="mine-amount">+{{ mineAmount }} ore</p>
      </div>

      <section class="upgrades-section">
        <h2>Upgrades</h2>
        <div class="upgrades-grid">
          <div 
            v-for="upgrade in upgrades" 
            :key="upgrade.id"
            class="upgrade-card"
            :class="{ disabled: ore < upgrade.cost }"
          >
            <h3>{{ upgrade.name }}</h3>
            <p class="description">{{ upgrade.description }}</p>
            <p class="cost">Cost: {{ Math.floor(upgrade.cost) }} ore</p>
            <p class="owned">Owned: {{ upgrade.owned }}</p>
            <button 
              @click="buyUpgrade(upgrade.id)"
              :disabled="ore < upgrade.cost"
              class="buy-btn"
            >
              Buy
            </button>
          </div>
        </div>
      </section>

      <!-- IN-APP STORE -->
      <section class="shop-section">
        <h2>🛍️ Cosmetics & Perks</h2>
        <div class="shop-grid">
          <div class="shop-item">
            <h3>Golden Pickaxe</h3>
            <p>✨ Premium cosmetic</p>
            <button @click="buyCosmetic('golden-pickaxe', 2.99)" class="shop-btn">$2.99</button>
          </div>
          <div class="shop-item">
            <h3>Faster Taps</h3>
            <p>⚡ +25% tap speed</p>
            <button @click="buyCosmetic('faster-taps', 1.99)" class="shop-btn">$1.99</button>
          </div>
          <div class="shop-item">
            <h3>Ad-Free Pass</h3>
            <p>🚫 Remove ads (7 days)</p>
            <button @click="buyCosmetic('ad-free', 0.99)" class="shop-btn">$0.99</button>
          </div>
          <div class="shop-item">
            <h3>Battle Pass</h3>
            <p>🎮 Seasonal rewards</p>
            <button @click="buyCosmetic('battle-pass', 4.99)" class="shop-btn">$4.99</button>
          </div>
        </div>
      </section>
    </main>

    <footer class="game-footer">
      <p>Kearns Mining Rig v2.0 Premium | Earn Real Bitcoin | Save Progress: {{ lastSaved }}</p>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      ore: 0,
      bankBalance: 0,
      mineAmount: 1,
      miningRate: 0,
      level: 1,
      lastSaved: new Date().toLocaleTimeString(),
      lastTransaction: '',
      
      // Premium System
      isPremium: false,
      premiumTier: null,
      premiumBoost: 0,
      nextBonusAmount: 5,
      showPremium: false,
      
      // Account System
      showAccount: false,
      totalEarnings: 0,
      monthlyEarnings: 0,
      referralEarnings: 0,
      referralCount: 0,
      referralLink: 'https://kearns-mining-rig.vercel.app?ref=USER123',
      btcWallet: '',
      withdrawAmount: 0,
      withdrawBalance: 0,
      oreValue: 0.001, // 1 ore = $0.001
      currentMonth: new Date().toLocaleString('default', { month: 'long' }),
      
      // Shop
      cosmetics: [],
      
      upgrades: [
        {
          id: 1,
          name: 'Better Pickaxe',
          description: 'Increase tap damage by 50%',
          cost: 10,
          owned: 0,
          costMultiplier: 1.15,
          effect: (owned) => {
            this.mineAmount = 1 + (owned * 0.5)
          }
        },
        {
          id: 2,
          name: 'Auto Miner',
          description: 'Mine +0.1 ore per second',
          cost: 100,
          owned: 0,
          costMultiplier: 1.2,
          effect: (owned) => {
            this.miningRate = owned * 0.1
          }
        },
        {
          id: 3,
          name: 'Ore Refinery',
          description: 'Double ore production',
          cost: 500,
          owned: 0,
          costMultiplier: 1.25,
          effect: (owned) => {
            if (owned > 0) {
              this.mineAmount *= 2
              this.miningRate *= 2
            }
          }
        },
        {
          id: 4,
          name: 'Drill Upgrade',
          description: 'Triple mining rate',
          cost: 2000,
          owned: 0,
          costMultiplier: 1.3,
          effect: (owned) => {
            if (owned > 0) {
              this.miningRate *= 3
            }
          }
        }
      ]
    }
  },
  computed: {
    totalOreValue() {
      return (this.ore + this.bankBalance) * this.oreValue
    }
  },
  methods: {
    mine() {
      let mineBonus = this.isPremium ? (1 + this.premiumBoost / 100) : 1
      this.ore += this.mineAmount * mineBonus
      this.monthlyEarnings += this.mineAmount * mineBonus * this.oreValue
      this.updateLevel()
      this.saveProgress()
    },
    depositOre() {
      if (this.ore > 0) {
        const depositAmount = Math.floor(this.ore)
        this.bankBalance += depositAmount
        this.ore = 0
        this.lastTransaction = `✅ Deposited ${depositAmount} ore to bank`
        this.updateLevel()
        this.saveProgress()
        this.clearTransactionMessage()
      }
    },
    withdrawOre() {
      if (this.bankBalance > 0) {
        const withdrawAmount = Math.floor(this.bankBalance)
        this.ore += withdrawAmount
        this.bankBalance = 0
        this.lastTransaction = `✅ Withdrew ${withdrawAmount} ore from bank`
        this.updateLevel()
        this.saveProgress()
        this.clearTransactionMessage()
      }
    },
    buyUpgrade(upgradeId) {
      const upgrade = this.upgrades.find(u => u.id === upgradeId)
      if (upgrade && this.ore >= upgrade.cost) {
        this.ore -= upgrade.cost
        upgrade.owned++
        upgrade.cost *= upgrade.costMultiplier
        upgrade.effect(upgrade.owned)
        this.updateLevel()
        this.saveProgress()
      }
    },
    selectPremium(tier) {
      this.isPremium = true
      this.premiumTier = tier
      this.showPremium = false
      
      if (tier === 'bronze') {
        this.premiumBoost = 50
        this.nextBonusAmount = 0
      } else if (tier === 'silver') {
        this.premiumBoost = 100
        this.nextBonusAmount = 5
      } else if (tier === 'gold') {
        this.premiumBoost = 200
        this.nextBonusAmount = 15
        this.referralCount += 0.3 // 30% referral bonus
      }
      
      this.lastTransaction = `✨ Premium Activated: ${tier.toUpperCase()}`
      this.saveProgress()
      this.clearTransactionMessage()
    },
    buyCosmetic(cosmetic, price) {
      this.monthlyEarnings -= price
      this.referralEarnings += price * 0.1 // 10% of each purchase goes to commissions
      this.cosmetics.push(cosmetic)
      this.lastTransaction = `🎁 Purchased ${cosmetic} for $${price}`
      this.saveProgress()
      this.clearTransactionMessage()
    },
    withdrawBitcoin() {
      if (this.withdrawAmount > 0 && this.btcWallet && this.withdrawAmount <= this.withdrawBalance) {
        this.totalEarnings += this.withdrawAmount
        this.withdrawBalance -= this.withdrawAmount
        this.lastTransaction = `✅ Withdrawal Pending: ${this.withdrawAmount} BTC to wallet`
        this.withdrawAmount = 0
        this.saveProgress()
        this.clearTransactionMessage()
        
        // In real app, call backend API here to process payment
        alert(`Bitcoin withdrawal request submitted!\n\n${this.withdrawAmount} BTC → ${this.btcWallet}\n\nProcessing in 24-48 hours`)
      }
    },
    copyReferral() {
      navigator.clipboard.writeText(this.referralLink)
      alert('Referral link copied!')
    },
    updateLevel() {
      this.level = Math.floor((this.ore + this.bankBalance) / 100) + 1
    },
    saveProgress() {
      this.lastSaved = new Date().toLocaleTimeString()
      const gameState = {
        ore: this.ore,
        bankBalance: this.bankBalance,
        mineAmount: this.mineAmount,
        miningRate: this.miningRate,
        level: this.level,
        upgrades: this.upgrades,
        isPremium: this.isPremium,
        premiumTier: this.premiumTier,
        totalEarnings: this.totalEarnings,
        monthlyEarnings: this.monthlyEarnings,
        referralEarnings: this.referralEarnings,
        referralCount: this.referralCount,
        cosmetics: this.cosmetics
      }
      localStorage.setItem('miningRigProgress', JSON.stringify(gameState))
    },
    loadProgress() {
      const saved = localStorage.getItem('miningRigProgress')
      if (saved) {
        const gameState = JSON.parse(saved)
        this.ore = gameState.ore || 0
        this.bankBalance = gameState.bankBalance || 0
        this.mineAmount = gameState.mineAmount || 1
        this.miningRate = gameState.miningRate || 0
        this.level = gameState.level || 1
        this.isPremium = gameState.isPremium || false
        this.premiumTier = gameState.premiumTier || null
        this.totalEarnings = gameState.totalEarnings || 0
        this.monthlyEarnings = gameState.monthlyEarnings || 0
        this.referralEarnings = gameState.referralEarnings || 0
        this.referralCount = gameState.referralCount || 0
        this.cosmetics = gameState.cosmetics || []
        if (gameState.upgrades) {
          this.upgrades = gameState.upgrades
        }
        this.lastTransaction = '📂 Progress loaded'
        this.clearTransactionMessage()
      }
      this.calculateWithdrawBalance()
    },
    calculateWithdrawBalance() {
      // Convert ore value to Bitcoin (needs real exchange rate in production)
      this.withdrawBalance = this.bankBalance * this.oreValue * 0.00001 // Simplified conversion
    },
    clearTransactionMessage() {
      setTimeout(() => {
        this.lastTransaction = ''
      }, 3000)
    }
  },
  mounted() {
    this.loadProgress()
    
    // Auto-mining loop
    setInterval(() => {
      let mineBonus = this.isPremium ? (1 + this.premiumBoost / 100) : 1
      this.ore += (this.miningRate / 10) * mineBonus
      this.monthlyEarnings += (this.miningRate / 10) * mineBonus * this.oreValue
      this.updateLevel()
    }, 100)

    // Periodic save
    setInterval(() => {
      this.saveProgress()
    }, 5000)
    
    // Daily bonus for premium users
    setInterval(() => {
      if (this.isPremium && this.premiumTier === 'silver') {
        this.monthlyEarnings += 5
        this.lastTransaction = '💰 Weekly $5 bonus received!'
        this.clearTransactionMessage()
      } else if (this.isPremium && this.premiumTier === 'gold') {
        this.monthlyEarnings += 15
        this.lastTransaction = '💰 Weekly $15 bonus received!'
        this.clearTransactionMessage()
      }
    }, 604800000) // 1 week
  }
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  font-family: 'Arial', sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  color: #333;
}

.game-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.game-header {
  background: rgba(0, 0, 0, 0.3);
  color: white;
  padding: 1.5rem;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

.header-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.game-header h1 {
  font-size: 2.5rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.header-buttons {
  display: flex;
  gap: 1rem;
}

.premium-btn, .account-btn {
  padding: 0.8rem 1.5rem;
  border: none;
  border-radius: 25px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
}

.premium-btn {
  background: linear-gradient(135deg, #ffd700 0%, #ffed4e 100%);
  color: #333;
}

.premium-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 20px rgba(255, 215, 0, 0.4);
}

.account-btn {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 2px solid white;
}

.account-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.subtitle {
  font-size: 1rem;
  opacity: 0.9;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background: white;
  border-radius: 15px;
  padding: 2rem;
  max-width: 800px;
  max-height: 80vh;
  overflow-y: auto;
  position: relative;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

.close-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  color: #999;
}

.modal h2 {
  margin-bottom: 1.5rem;
  color: #333;
}

.premium-tiers {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.tier {
  border: 2px solid #ddd;
  border-radius: 10px;
  padding: 1.5rem;
  text-align: center;
  transition: all 0.3s;
}

.tier.featured {
  border-color: #ffd700;
  background: rgba(255, 215, 0, 0.1);
  transform: scale(1.05);
}

.tier h3 {
  margin-bottom: 0.5rem;
  color: #333;
}

.price {
  font-size: 1.5rem;
  font-weight: bold;
  color: #667eea;
  margin-bottom: 1rem;
}

.tier ul {
  list-style: none;
  margin-bottom: 1.5rem;
  text-align: left;
}

.tier li {
  padding: 0.5rem 0;
  color: #666;
}

.account-stats {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1rem;
  margin-bottom: 2rem;
}

.stat-box {
  background: #f5f5f5;
  padding: 1rem;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.stat-box span:first-child {
  color: #999;
  font-size: 0.9rem;
}

.amount {
  font-size: 1.3rem;
  font-weight: bold;
  color: #667eea;
}

.withdrawal-section, .referral-section {
  margin-bottom: 2rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid #eee;
}

.withdrawal-section h3, .referral-section h3 {
  margin-bottom: 1rem;
  color: #333;
}

.withdrawal-section input, .referral-link input {
  width: 100%;
  padding: 0.8rem;
  margin-bottom: 1rem;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
}

.referral-link {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.referral-link input {
  margin-bottom: 0;
}

.copy-btn {
  padding: 0.8rem 1.5rem;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
}

.copy-btn:hover {
  background: #764ba2;
}

.referral-count {
  text-align: center;
  color: #667eea;
  font-weight: bold;
}

.game-main {
  flex: 1;
  padding: 2rem;
  overflow-y: auto;
}

.premium-banner {
  background: linear-gradient(135deg, #ffd700 0%, #ffed4e 100%);
  color: #333;
  padding: 1rem;
  border-radius: 10px;
  margin-bottom: 1.5rem;
  font-weight: bold;
  text-align: center;
}

.ad-container {
  background: rgba(0, 0, 0, 0.1);
  border: 2px dashed rgba(255, 255, 255, 0.5);
  color: white;
  padding: 1rem;
  border-radius: 10px;
  margin-bottom: 1.5rem;
  text-align: center;
}

.stats-panel {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
  background: white;
  border-radius: 10px;
  padding: 1.5rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.stat {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.stat-label {
  font-size: 0.85rem;
  color: #666;
  font-weight: bold;
}

.stat-value {
  font-size: 1.6rem;
  font-weight: bold;
  color: #667eea;
}

.banking-area {
  background: white;
  border-radius: 10px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  border-left: 4px solid #f5576c;
}

.bank-controls {
  display: flex;
  gap: 1rem;
  margin-bottom: 1rem;
}

.bank-btn {
  flex: 1;
  padding: 1rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
  color: white;
}

.deposit-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.deposit-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 15px rgba(102, 126, 234, 0.4);
}

.withdraw-btn {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.withdraw-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 15px rgba(245, 87, 108, 0.4);
}

.bank-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.bank-info {
  font-size: 0.95rem;
  color: #667eea;
  font-weight: bold;
  text-align: center;
}

.mining-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 2rem 0;
  gap: 1rem;
}

.mine-button {
  width: 140px;
  height: 140px;
  border-radius: 50%;
  border: none;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
  font-size: 2.5rem;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 8px 20px rgba(245, 87, 108, 0.4);
  transition: transform 0.1s, box-shadow 0.1s;
}

.mine-button:hover {
  transform: scale(1.08);
  box-shadow: 0 12px 30px rgba(245, 87, 108, 0.6);
}

.mine-button:active {
  transform: scale(0.92);
}

.mine-amount {
  font-size: 1.3rem;
  font-weight: bold;
  color: white;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.3);
}

.upgrades-section, .shop-section {
  background: white;
  border-radius: 10px;
  padding: 2rem;
  margin-bottom: 2rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.upgrades-section h2, .shop-section h2 {
  margin-bottom: 1.5rem;
  color: #333;
  font-size: 1.6rem;
}

.upgrades-grid, .shop-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(230px, 1fr));
  gap: 1.5rem;
}

.upgrade-card, .shop-item {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  border-radius: 10px;
  padding: 1.5rem;
  border: 2px solid #ddd;
  transition: all 0.3s;
  text-align: center;
}

.upgrade-card:hover:not(.disabled), .shop-item:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
  border-color: #667eea;
}

.upgrade-card.disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.upgrade-card h3, .shop-item h3 {
  margin-bottom: 0.5rem;
  color: #333;
}

.description {
  font-size: 0.85rem;
  color: #666;
  margin-bottom: 1rem;
}

.cost {
  font-weight: bold;
  color: #f5576c;
  margin-bottom: 0.5rem;
}

.owned {
  font-size: 0.85rem;
  color: #667eea;
  margin-bottom: 1rem;
}

.buy-btn, .shop-btn {
  width: 100%;
  padding: 0.8rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
}

.buy-btn:hover:not(:disabled), .shop-btn:hover {
  transform: scale(1.02);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.buy-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.featured-btn {
  background: linear-gradient(135deg, #ffd700 0%, #ffed4e 100%);
  color: #333;
}

.featured-btn:hover {
  box-shadow: 0 4px 12px rgba(255, 215, 0, 0.4);
}

.game-footer {
  background: rgba(0, 0, 0, 0.3);
  color: white;
  padding: 1rem;
  text-align: center;
  font-size: 0.85rem;
}

.warning {
  color: #f5576c;
  font-weight: bold;
  margin-bottom: 1rem;
}

@media (max-width: 768px) {
  .header-top {
    flex-direction: column;
    gap: 1rem;
  }

  .game-header h1 {
    font-size: 1.8rem;
  }

  .header-buttons {
    width: 100%;
  }

  .premium-btn, .account-btn {
    flex: 1;
  }

  .mine-button {
    width: 110px;
    height: 110px;
    font-size: 2rem;
  }

  .modal {
    width: 95%;
    max-height: 90vh;
  }

  .account-stats {
    grid-template-columns: 1fr;
  }

  .bank-controls {
    flex-direction: column;
  }

  .upgrades-grid, .shop-grid {
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  }
}
</style>
