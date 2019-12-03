<template>
  <div id="app">
    <menu-component />
  </div>
</template>

<script>
import MenuComponent from "./components/Menu";

export default {
  name: "App",
  components: {
    MenuComponent
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
