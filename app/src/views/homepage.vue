<template>
  <div class="game">
    <div class="shop" v-if="shopopen">
      <div v-for="item in shop" :key="item.item" class="shop-item">
        <h2>{{ item.item }}</h2>
        <p>{{ item.effect }}</p>
        <p>You have {{ boosts[item.item] }}</p>
        <p>Price: ${{ item.price.toFixed(2) }}</p>
        <button @click="buyItem(item)">Buy</button>
      </div>
    </div>
    <button class="toggle-shop" @click="shopopen = !shopopen">{{ shopopen ? "Close Shop" : "Open Shop" }}</button>
    <div class="stats">
      <h3>Money: ${{ money.toFixed(2) }}</h3>
      <h3>Money Gain: ${{(((1+boosts['Bounty Multiplier'])*(1.1**boosts['Greater Yield']))**((1+boosts['Gold Transmuter']/100)*(1.01^boosts["Hyperinflation Machine"])))**(1+boosts['Galactic Threats']/100)}}</h3>
      <h3>Formula: ( ( ({{boosts["Bounty Multiplier"]}} BM + 1) * (1.1 ^ {{boosts["Greater Yield"]}} GY) ) ^ ( ({{boosts["Gold Transmuter"]}} GoT / 100 + 1) * (1.01 ^ {{boosts["Hyperinflation Machine"]}} HM) ) ) ^ ({{boosts["Galactic Threats"]}} GaT / 20 + 1)</h3>
      <h3>Enemy Level: {{ currentenemystats.level }}</h3>
    </div>
    <div class="collect">Drop Corpse Here</div>
    <button class="enemy" draggable="true" @dragstart="onDragStart" @click="damageEnemy":style="{ left: enemyPosition.x + 'px', top: enemyPosition.y + 'px', width: ((3*boosts['Shot Radius Augments'] + 65)*(1-boosts['Galactic Threats']/100)) + 'px', height: ((3*boosts['Shot Radius Augments'] + 35)*(1-boosts['Galactic Threats']/100)) + 'px'}">
      <div class="hp-bar">
        <div class="hp-fill":style="{ width: (currentenemystats.health / currentenemystats.maxhealth) * 100 + '%' }" ></div>
      </div>
      <div v-if="!currentenemystats.dead">HP: {{ currentenemystats.health }}/{{ currentenemystats.maxhealth }}</div>
      <div v-else>Drag to Collect</div>
    </button>
  </div>
</template>
<script setup>
import { ref, reactive, onMounted } from 'vue'
const shopopen = ref(false)
const money = ref(999999999999999)
const shop = reactive([
  {
    item: 'Bounty Multiplier',
    effect: "Increases base money gain by 1",
    price: 5,
    max: 999999
  },
  {
    item: 'Greater Yield',
    effect: "Increases money gain by 1.1x compounding",
    price: 8,
    max: 999999
  },
  {
    item: 'Stronger Ammunition',
    effect: "Deal 1 extra damage per shot",
    price: 10,
    max: 999999
  },
  {
    item: 'Tranquilizer Improvements',
    effect: "Limits the enemy's movement area by +1%",
    price: 15,
    max: 45
  },
  {
    item: 'Shot Radius Augments',
    effect: "Increases enemy size",
    price: 25,
    max: 50
  },
  {
    item: 'Gold Transmuter',
    effect: "Increases money gain by +^0.01",
    price: 100000,
    max: 99999999999999999999999999999999999999999
  },
  {
    item: 'Hyperinflation Machine',
    effect: "Increases exponential money gain by 1.01x compounding",
    price: 10000000,
    max: 50
  },
  {
    item: 'Galactic Threats',
    effect: "Increases resulting money gain by +^0.01 at the cost of increasing enemy health scaling by 10% and decreasing enemy size by 1%",
    price: 10000000,
    max: 50
  },
])
const boosts = reactive({
  "Bounty Multiplier": 0,
  "Greater Yield": 0,
  "Stronger Ammunition": 0,
  "Tranquilizer Improvements":0,
  "Shot Radius Augments":0,
  "Gold Transmuter":0,
  "Hyperinflation Machine":0,
  "Galactic Threats":0
})
const currentenemystats = reactive({
  health: 1,
  maxhealth: 1,
  dead: false,
  level: 1,
})
const enemyPosition = reactive({
  x: window.innerWidth / 2,
  y: window.innerHeight / 2
})

function moveEnemy() {
  if (currentenemystats.dead){return}
  const padding = 200
  const maxX = window.innerWidth - padding
  const maxY = window.innerHeight - padding
  const minX = maxX * (boosts["Tranquilizer Improvements"]/100)
  const minY = maxY * (boosts["Tranquilizer Improvements"]/100)
  enemyPosition.x = ((maxX-2*minX)*Math.random())+minX
  enemyPosition.y = ((maxY-2*minY)*Math.random())+minY
}

function damageEnemy() {
  if (currentenemystats.dead){return}
  currentenemystats.health -= (1 + boosts["Stronger Ammunition"])
  if (currentenemystats.health <= 0) {
    currentenemystats.dead = true
    return
  }
  moveEnemy()
}
function respawnEnemy() {
  currentenemystats.level++
  currentenemystats.maxhealth = Math.floor(1+(1.01 ** currentenemystats.level)*(1+(boosts["Galactic Threats"]/10)**currentenemystats.level))
  currentenemystats.health = currentenemystats.maxhealth
  currentenemystats.dead = false

  moveEnemy()
}
function collectReward() {
  // debug obv it wont be 100000000000000 money
  const gain = (((1+boosts['Bounty Multiplier'])*(1.1**boosts['Greater Yield']))**((1+boosts['Gold Transmuter']/100)*(1.01^boosts["Hyperinflation Machine"])))**(1+boosts['Galactic Threats']/20)
  money.value += gain
  money.value = parseFloat(money.value.toFixed(2))
  respawnEnemy()
}
function buyItem(item) {
  if (money.value >= item.price && boosts[item.item] < item.max) {
    money.value -= item.price
    boosts[item.item]++
    item.price *= 1.25
  }
}
function onDragStart(event) {
  if (!currentenemystats.dead) {
    event.preventDefault()
  }
}
onMounted(() => {
  const zone = document.querySelector('.collect')
  zone.addEventListener('dragover', e => {
    e.preventDefault()
  })
  zone.addEventListener('drop', e => {
    e.preventDefault()
    if (currentenemystats.dead) {
      collectReward()
    }
  })
  moveEnemy()
})
</script>
<!-- I DID NOT MAKE ANY OF THIS CSS, I ONLY DID BACKEND -->
<style scoped>
.game {
  font-family: Arial, sans-serif;
  text-align: center;
  background: #111;
  color: white;
  padding: 20px;
  position: relative;
}
.shop {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}
.shop-item {
  background: #222;
  padding: 15px;
  border-radius: 10px;
  width: 200px;
  box-shadow: 0 0 10px #000;
}
.shop-item button {
  background: #4CAF50;
  border: none;
  padding: 8px;
  color: white;
  border-radius: 5px;
}
.shop-item button:hover {
  background: #45a049;
}
.toggle-shop {
  margin-bottom: 15px;
  padding: 8px 15px;
  background: #333;
  border: none;
  color: white;
  border-radius: 5px;
}
.stats {
  margin: 15px;
}
.enemy {
  position: absolute;
  border-radius: 12px;
  background: #8b0000;
  border: none;
  color: white;
  transition: left 0.25s ease, top 0.25s ease, transform 0.1s;
}
.enemy:hover {
  transform: scale(1.05);
}
.hp-bar {
  position: absolute;
  top: -15px;
  left: 0;
  width: 100%;
  height: 10px;
  background: #444;
  border-radius: 5px;
  overflow: hidden;
}
.hp-fill {
  height: 100%;
  background: limegreen;
  transition: width 0.2s;
}
.collect {
  position: fixed;
  bottom: 20px;
  right: 20px;
  padding: 20px;
  border: 2px dashed gold;
  border-radius: 12px;
  background: #222;
}
</style>