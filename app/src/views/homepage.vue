<template>
    <div>
        <div>
            <div v-for="item in shop" :key="item.item">
                <h2>{{ item.item }}</h2>
                <p>{{ item.effect }}</p>
                <p>You have {{ boosts[item.item] }} of this item.</p>
                <p>Price: ${{ item.price }}</p>
                <button @click="buyItem(item.item)">Buy</button>
            </div>
        </div>
        <h3>Money: ${{ money }}</h3>
        <button id="target" @click="relocate">hi</button>
    </div>
</template>

<script setup>
    import { ref, reactive } from 'vue'
    const shop = reactive([
        {
            item: 'Bounty Multiplier',
            effect: "Increases base money gain by 1",
            img: '',
            price: 10
        },
        {
            item: 'Shot Size',
            effect: "Increases area of click by 1, Max of 20",
            img: '',
            price: 10
        },
        {
            item: 'Greater Yield',
            effect: "Increases money gain by 1.5x compounding",
            img: '',
            price: 10
        }
    ])
    const boosts = reactive({
        "Bounty Multiplier": 0,
        "Shot Size": 0,
        "Greater Yield": 0
    })
    const money = ref(0)
    function clamp(min, max, value) {
    return Math.min(Math.max(value, min), max);
    }
    function relocate(){
        money.value += Number(((1+boosts["Bounty Multiplier"])*(1.5**boosts["Greater Yield"])).toFixed(2));
        const button = document.getElementById('target');
        const max = [window.innerWidth  - button.offsetWidth,window.innerHeight - button.offsetHeight];
        const newpos = [Math.round(Math.random() * 250)*((-1)**Math.ceil(Math.random() * 4)),Math.round(Math.random() * 250)*((-1)**Math.ceil(Math.random() * 4))]
        //button.style.transform = `translate(${clamp(0,max[0],newpos[0]+max[0]/2)}px, ${clamp(0,max[1],newpos[1]+max[1]/2)}px)`;
    }
    function buyItem(item){
        const index = shop.find(i => i.item === item);
        if(money.value >= index.price){
            money.value -= index.price;
            boosts[index.item] += 1;
        }
    }
</script>

<style scoped>

</style>