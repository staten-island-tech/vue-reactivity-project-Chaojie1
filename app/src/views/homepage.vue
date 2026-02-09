<template>
    <div>
        <button @click="relocate">hi</button>
    </div>
</template>

<script setup>
    import { ref } from 'vue'
    function clamp(min, max, value) {
    return Math.min(Math.max(value, min), max);
    }
    function relocate(){
        const button = document.body.querySelector('button');
        const max = [window.innerWidth  - button.offsetWidth,window.innerHeight - button.offsetHeight];
        const newpos = [Math.round(Math.random() * 250)*((-1)**Math.ceil(Math.random() * 4)),Math.round(Math.random() * 250)*((-1)**Math.ceil(Math.random() * 4))]
        button.style.transform = `translate(${clamp(0,max[0],newpos[0]+max[0]/2)}px, ${clamp(0,max[1],newpos[1]+max[1]/2)}px)`;
    }
</script>

<style scoped>

</style>