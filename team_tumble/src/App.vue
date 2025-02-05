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
  font-family: Lilita-One, Pacifico;
  margin: 0;
  padding: 0;
  background-color: #373434;
  min-width: 1350px;
  min-height: 830px;
  height: 100vh;
}

h1 {
  margin: 0;
  font-size: 54px;
  color: #0AC6C6;
}
</style>
