<script setup>
import Calculator from './components/Calculator.vue'
import Explanation from './components/Explanation.vue'
</script>

<template>
  <header>
  <Explanation/>
  </header>

  <main>
    <Calculator/>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
  header{
    margin-right: 2em;
  }
}
</style>
