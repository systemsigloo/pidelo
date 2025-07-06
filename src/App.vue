<template>
  <div style="max-width: 800px; margin: auto; padding: 2px;">
    <h1 style="padding-left: 25px;">Good Friends</h1>
    <Catalog />
  </div>
</template>

<script setup>
import Catalog from './components/Catalog.vue';
const apiUrl = import.meta.env.VITE_API_URL;
</script>
<style>

@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap');

body {
    font-family: 'Poppins', sans-serif;
    background-color: rgb(108, 1, 1); 
    color: #ba9330;
}
</style>

