<template>
  <div id="playlist">
    <div class="dropdown"></div>
    <h1>Playlist</h1>
    <ul>
      <li
        v-for="(song,index) in songList"
        @click.stop="onPlaylistClick(index)"
        :key="song.title"
        :class="{playing:current==index}"
      >
        <span class="index">{{index+1}}.</span>
        {{ song.author }} - {{ song.title }}
      </li>
      <li>Lorem, ipsum dolor.</li>
      <li>Voluptatum, minima iusto.</li>
      <li>Suscipit, sequi architecto!</li>
      <li>Minima, natus possimus.</li>
      <li>Ipsam, incidunt obcaecati?</li>
      <li>Iste, quasi est!</li>
    </ul>
  </div>
</template>

<script>
export default {
  props: ["songList", "current"],
  data() {
    return {};
  },
  mounted() {},
  methods: {
    onPlaylistClick(index) {
      this.$emit("clickPlaylist", index);
    }
  }
};
</script>
<style scoped>
div.dropdown {
  margin: 10px auto;
  width: 60px;
  height: 20px;
  background: url("../assets/icons/dropdown.svg") no-repeat center left;
  background-size: 60px 60px;
}

#playlist h1 {
  text-align: center;
}

ul {
  text-align: center;
  list-style: none;
  padding: 0px;
}

ul li {
  cursor: pointer;
  padding: 10px 0 10px 0px;
}

span.index {
  padding-right: 3px;
}

.playing {
  background-color: wheat;
}
</style>