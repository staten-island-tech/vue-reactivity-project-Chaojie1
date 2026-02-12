<template>
    <div>
        <h2>{{ item.item }}</h2>
        <p>{{ item.effect }}</p>
        <p>You have {{ boosts[item.item] }} of this item.</p>
        <p>Price: ${{ item.price }}</p>
        <button @click="buyItem(item.item)">Buy</button>
    </div>
</template>

<script setup>
import { buyItem } from '@/views/homepage.vue'
    defineProps({
        item:{
            type: Object,
            required: true
        },
        boosts:{
            type: Array,
            required: true
        }
    })
</script>

<style scoped>

</style>