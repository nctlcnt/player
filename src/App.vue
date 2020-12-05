<template>
  <div id="app" ref="app">
    <player></player>
  </div>
</template>

<script>
import Player from "./components/Player.vue";

export default {
  name: "App",
  components: {
    Player,
  },
};
</script>

<style>
body,
html {
  padding: 0px;
  margin: 0px;
  overscroll-behavior-y: contain;
}
#app {
  text-align: center;
  height: 100vh;
  overflow: hidden;
  margin: 0px;
  padding: 0px;
  max-width: 600px;
  /* background-color: black; */
}
#player {
  position: absolute;
  top: 0px;
  left: 0px;
}
</style>
