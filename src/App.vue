<template>
  <div>
    <Three></Three>
  </div>
</template>

<script >

import Three from './components/ThreeComponent.vue'
export default {
  name: 'App',
  components: {
    Three
  },
}

</script>

<style>
body{
  margin:0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

}
</style>
