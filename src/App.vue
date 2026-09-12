<template>
  <div id="app" class="game-container">
    <header class="game-header">
      <h1>⛏️ Kearns Mining Rig</h1>
      <p class="subtitle">Tap to Mine</p>
    </header>

    <main class="game-main">
      <div class="stats-panel">
        <div class="stat">
          <span class="stat-label">Ore Mined:</span>
          <span class="stat-value">{{ Math.floor(ore) }}</span>
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
    </main>

    <footer class="game-footer">
      <p>Kearns Mining Rig v1.0 | Save Progress: {{ lastSaved }}</p>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      ore: 0,
      mineAmount: 1,
      miningRate: 0,
      level: 1,
      lastSaved: new Date().toLocaleTimeString(),
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
  methods: {
    mine() {
      this.ore += this.mineAmount
      this.updateLevel()
      this.saveProgress()
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
    updateLevel() {
      this.level = Math.floor(this.ore / 100) + 1
    },
    saveProgress() {
      this.lastSaved = new Date().toLocaleTimeString()
      // TODO: Implement localStorage save
    }
  },
  mounted() {
    // Auto-mining loop
    setInterval(() => {
      this.ore += this.miningRate / 10
      this.updateLevel()
    }, 100)

    // Periodic save
    setInterval(() => {
      this.saveProgress()
    }, 5000)
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
  padding: 2rem;
  text-align: center;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

.game-header h1 {
  font-size: 3rem;
  margin-bottom: 0.5rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.subtitle {
  font-size: 1.2rem;
  opacity: 0.9;
}

.game-main {
  flex: 1;
  padding: 2rem;
  overflow-y: auto;
}

.stats-panel {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
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
  font-size: 0.9rem;
  color: #666;
  font-weight: bold;
}

.stat-value {
  font-size: 1.8rem;
  font-weight: bold;
  color: #667eea;
}

.mining-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 3rem 0;
  gap: 1rem;
}

.mine-button {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  border: none;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
  font-size: 2rem;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 8px 20px rgba(245, 87, 108, 0.4);
  transition: transform 0.1s, box-shadow 0.1s;
}

.mine-button:hover {
  transform: scale(1.05);
  box-shadow: 0 10px 25px rgba(245, 87, 108, 0.6);
}

.mine-button:active {
  transform: scale(0.95);
}

.mine-amount {
  font-size: 1.5rem;
  font-weight: bold;
  color: white;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.3);
}

.upgrades-section {
  background: white;
  border-radius: 10px;
  padding: 2rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.upgrades-section h2 {
  margin-bottom: 1.5rem;
  color: #333;
  font-size: 1.8rem;
}

.upgrades-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1.5rem;
}

.upgrade-card {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  border-radius: 10px;
  padding: 1.5rem;
  border: 2px solid #ddd;
  transition: all 0.3s;
  cursor: pointer;
}

.upgrade-card:hover:not(.disabled) {
  transform: translateY(-5px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
  border-color: #667eea;
}

.upgrade-card.disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.upgrade-card h3 {
  margin-bottom: 0.5rem;
  color: #333;
}

.description {
  font-size: 0.9rem;
  color: #666;
  margin-bottom: 1rem;
}

.cost {
  font-weight: bold;
  color: #f5576c;
  margin-bottom: 0.5rem;
}

.owned {
  font-size: 0.9rem;
  color: #667eea;
  margin-bottom: 1rem;
}

.buy-btn {
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

.buy-btn:hover:not(:disabled) {
  transform: scale(1.02);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.buy-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.game-footer {
  background: rgba(0, 0, 0, 0.3);
  color: white;
  padding: 1rem;
  text-align: center;
  font-size: 0.9rem;
}

@media (max-width: 768px) {
  .game-header h1 {
    font-size: 2rem;
  }

  .mine-button {
    width: 100px;
    height: 100px;
    font-size: 1.5rem;
  }

  .upgrades-grid {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  }
}
</style>
