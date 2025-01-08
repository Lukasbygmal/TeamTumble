<template>
  <div id="app">
    <h1>Team Tumble</h1>
    <PlinkoBoard />
  </div>
</template>

<script>
import PlinkoBoard from './components/Layout.vue';

export default {
  name: 'App',
  components: {
    PlinkoBoard
  }
};
</script>

<style>
#app {
  text-align: center;
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}
h1 {
  margin: 20px 0;
}
</style>
