# Kearns Mining Rig ⛏️

A fun tap-to-mine incremental/idle game built with Vue 3 and Vite.

## Features

- **Tap to Mine**: Click the big button to mine ore
- **Auto Mining**: Purchase upgrades to earn ore passively
- **Progression**: Level up as you accumulate ore
- **Upgrade System**: Buy pickaxes, auto-miners, refineries, and drills
- **Progressive Scaling**: Each upgrade costs more but gets exponentially better

## Getting Started

### Prerequisites

- Node.js (v16 or higher)
- npm or yarn

### Installation

1. Clone the repository:
```bash
git clone https://github.com/KDAWG6988/kearns-mining-rig.git
cd kearns-mining-rig
```

2. Install dependencies:
```bash
npm install
```

3. Start the development server:
```bash
npm run dev
```

4. Open your browser to `http://localhost:5173`

## Available Scripts

- `npm run dev` - Start the development server
- `npm run build` - Build for production
- `npm run preview` - Preview the production build

## Gameplay

### How to Play

1. **Mine Ore**: Click the large diamond button to mine ore
2. **Buy Upgrades**: Use mined ore to purchase upgrades in the Upgrades section
3. **Increase Production**: Each upgrade increases your mining rate or tap damage
4. **Level Up**: Earn levels as you progress (1 level per 100 ore)
5. **Optimize**: Find the best upgrade strategy to maximize your ore production

### Upgrades

- **Better Pickaxe**: Increases tap damage by 50% per upgrade
- **Auto Miner**: Adds 0.1 ore/second per upgrade
- **Ore Refinery**: Doubles all ore production
- **Drill Upgrade**: Triples mining rate

## Project Structure

```
kearns-mining-rig/
├── src/
│   ├── main.js           # Vue app entry point
│   └── App.vue           # Main game component
├── index.html            # HTML entry point
├── package.json          # Project dependencies
├── vite.config.js        # Vite configuration
├── .gitignore            # Git ignore rules
└── README.md             # This file
```

## Technology Stack

- **Vue 3**: Progressive JavaScript framework
- **Vite**: Next generation frontend tooling
- **JavaScript**: Game logic and state management

## Future Enhancements

- [ ] LocalStorage save/load system
- [ ] Multiple mining operations
- [ ] Prestige/reset mechanic
- [ ] Leaderboard
- [ ] Achievements
- [ ] Sound effects
- [ ] Mobile optimization
- [ ] Dark mode

## License

MIT

## Author

KDAWG6988

---

Happy Mining! 🎮⛏️
