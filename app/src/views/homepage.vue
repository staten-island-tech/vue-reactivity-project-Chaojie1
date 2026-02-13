<template>
    <div>
         <div>
            <div v-if="shopopen" v-for="item in shop" :key="item.item">
                <h2>{{ item.item }}</h2>
                <p>{{ item.effect }}</p>
                <p>You have {{ boosts[item.item] }} of this item.</p>
                <p>Price: ${{ item.price }}</p>
                <button @click="buyItem(item.item)">Buy</button>
            </div>
        </div>
        <button v-if="shopopen" @click="shopopen = !shopopen">Close Shop</button>
        <button v-else @click="shopopen = !shopopen">Open Shop</button>
        <h3>Money: ${{ money }}</h3>
        <h3>Money Gain: ${{ ((1+boosts["Bounty Multiplier"])*(1.1**boosts["Greater Yield"])).toFixed(2) }}</h3>
        <button id="target" @click="relocate">hp: 3/3</button>
    </div>
</template>

<script setup>
    import { ref, reactive } from 'vue'
    const shop = reactive([
        {
            item: 'Bounty Multiplier',
            effect: "Increases base money gain by 1",
            img: '',
            price: 10,
            max: 100000000
        },
        {
            item: 'Shot Size',
            effect: "Increases area of click by 1, Max of 20",
            img: '',
            price: 10,
            max: 20
        },
        {
            item: 'Greater Yield',
            effect: "Increases money gain by 1.1x compounding",
            img: '',
            price: 10,
            max: 100000000
        },
        {
            item: 'Stronger Ammunition',
            effect: "Deal 1 extra damage per shot",
            img: '',
            price: 10,
            max: 100000000
        }
    ])
    const boosts = reactive({
        "Bounty Multiplier": 0,
        "Shot Size": 0,
        "Greater Yield": 0,
        "Stronger Ammunition": 0
    })
    const money = ref(0)
    function clamp(min, max, value) {
    return Math.min(Math.max(value, min), max);
    }
    function relocate(){
        const button = document.getElementById('target');
        if (currentenemystats.dead){
            money.value += ((1+boosts["Bounty Multiplier"])*(1.1**boosts["Greater Yield"]))
            money.value = parseFloat(money.value.toFixed(2));
            currentenemystats.dead = false;
            currentenemystats.health = currentenemystats.maxhealth;
            button.textContent = `hp: ${currentenemystats.health}/${currentenemystats.maxhealth}`;
        } else {
            currentenemystats.health -= (1 + boosts["Stronger Ammunition"]);
            button.textContent = `hp: ${currentenemystats.health}/${currentenemystats.maxhealth}`;
        }
        if (currentenemystats.health <= 0) {
            currentenemystats.dead = true;
            button.textContent = "collect corpse";
            return
        }
        const max = [window.innerWidth  - button.offsetWidth,window.innerHeight - button.offsetHeight];
        const newpos = [Math.round(Math.random() * 250)*((-1)**Math.ceil(Math.random() * 4)),Math.round(Math.random() * 250)*((-1)**Math.ceil(Math.random() * 4))]
        button.style.transform = `translate(${clamp(0,max[0],newpos[0]+max[0]/2)}px, ${clamp(0,max[1],newpos[1]+max[1]/2)}px)`;
    }
    function buyItem(item){
        const index = shop.find(i => i.item === item);
        if(money.value >= index.price){
            money.value -= index.price;
            boosts[index.item] += 1;
        }
    }
    const shopopen = ref(false);
    const currentenemystats = reactive({
        health: 3,
        maxhealth: 3,
        dead: false
    })
</script>

<style scoped>
</style>