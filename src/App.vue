<script setup lang="ts">
import { defineComponent } from 'vue'
import Timer from './components/Timer.vue'

defineComponent({
  components:{
    Timer
  }
})

</script>

<template>
  <v-app>
      <Timer />
  </v-app>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: rgb(60, 60, 40);
}

</style>