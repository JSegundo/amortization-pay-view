<template>
  <AmortizationView />
</template>

<script>
import AmortizationView from "./components/amortizationView.vue"

export default {
  name: "App",
  components: {
    AmortizationView,
  },
}
</script>

<style>
#app {
  font-family: "Poppins", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
}
</style>
