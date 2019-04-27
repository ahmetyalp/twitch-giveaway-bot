<template>
  <div id="app">
    <GiveAway/>
  </div>
</template>

<script>
import GiveAway from "./components/GiveAway";

export default {
  name: "app",
  components: {
    GiveAway
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  margin-left: 60px;
}
</style>
