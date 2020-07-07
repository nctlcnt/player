<template>
  <div id="app">
    <player @updatetimer="sendTimer"></player>
    <lyrics v-bind:timestamp="timer"></lyrics>
  </div>
</template>

<script>
import Player from "./components/Player.vue";
import Lyrics from "./components/Lyrics.vue";

export default {
  name: "App",
  components: {
    Player,
    Lyrics,
  },
  data() {
    return {
      timer: 0,
    };
  },
  methods: {
    sendTimer(val) {
      this.timer = val;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
