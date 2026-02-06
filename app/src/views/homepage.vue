<template>
    <div>
        <button @click="relocate"></button>
    </div>
</template>

<script setup>
    import { ref } from 'vue'
    function relocate(){
        document.querySelector('button').classList = `absolute top-${Math.round(Math.random)} left-0`
    }
</script>

<style scoped>

</style>