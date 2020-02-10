<template>
  <div id="app">
    <Machine />
  </div>
</template>

<script>
import Machine from './components/Machine.vue'

export default {
  name: 'App',
  components: {
    Machine
  }
}
</script>

<style>
#app {
  font-family: 'Consolas', 'Courier New', Courier, monospace;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
