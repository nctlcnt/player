<template>
  <div class="player-view" ref="app">
    <div
      id="player"
      :style="playerStyle"
      @mousemove="updateProgressDragging"
      @mouseup="stopDragging"
    >
      <header></header>
      <audio :src="this.songs[this.curIndex].url"></audio>
      <img
        :src="require('../assets/covers/'+this.songs[this.curIndex].cover)"
        :style="{height:coverHeight+'px',width:coverWidth+'px'}"
        alt="cover"
        class="cover"
      />
      <span class="detail">{{songs[curIndex].author}} - {{songs[curIndex].title}}</span>
      <div id="progress-container" @click="updateProgress" @mousedown.stop="startDragging">
        <div class="progress" :style="{ width:prog+'px' }" ref="progress"></div>
      </div>
      <span class="now">{{ currentTime | timeformatter }}</span>
      <span class="all">{{ duration | timeformatter}}</span>
      <div class="button-container">
        <button class="round prev" @click="prev"></button>
        <button class="play" v-if="!playing" @click="play"></button>
        <button class="stop" v-else @click="pause"></button>
        <button class="round next" @click="next"></button>
      </div>
    </div>

    <div id="playlist">
      <div class="dropdown"></div>
      <h1>Playlist</h1>
      <ul>
        <li
          v-for="(song,index) in songs"
          @click="onPlaylistClick(index)"
          :key="song.title"
          :class="{playing:curIndex==index}"
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
  </div>
</template>

<script>
export default {
  data() {
    return {
      all: 0,
      prog: 0,
      timer: null,
      currentTime: 0,
      duration: 0,
      cur: new Audio(),
      songs: [
        {
          song: new Audio(),
          author: "花澤香菜 (はなざわ かな)",
          title: "恋愛サーキュレーション (恋爱循环) (《化物语》TV动画片头曲) ",
          cover: "cover3.jpg",
          playing: false,
          url: require("../assets/songs/123.mp3")
        },
        {
          song: new Audio(),
          author: "Hatamoto Hiro",
          title: "first love",
          cover: "cover.jpg",
          playing: false,
          url: require("../assets/songs/Far_memories.mp3")
        },
        {
          song: new Audio(),
          author: "risou",
          title: "far memories",
          cover: "cover2.jpg",
          playing: false,
          url: require("../assets/songs/first.mp3")
        }
      ],
      curIndex: 0,
      playing: false,
      isScrolled: false,
      playerStyle: {
        opacity: 1
      },
      dragging: false,
      coverHeight: 0,
      coverWidth: 0,
      song: null,
      loading: true,
      loadingStyle: {
        height: 0,
        width: 0,
        top: 0,
        left: 30 + "px"
      },
      windowHeight: 0,
      windowWidth: 0,
      // touch event
      startX: 0,
      startY: 0,
      x: 0,
      y: 0,
      showPlaylist: false,
      show: false
    };
  },
  mounted() {
    this.all = document.getElementById("progress-container").clientWidth;
    this.windowHeight = window.innerHeight;
    this.windowWidth = window.innerWidth;
    this.height = this.windowHeight / 1.5;

    // the loading filter
    let cover = document.querySelector("img.cover");
    this.loadingStyle.width = this.coverWidth + "px";
    this.loadingStyle.height = this.coverWidth + "px";
    this.loadingStyle.left = cover.offsetLeft + "px";
    this.loadingStyle.top = cover.offsetTop + "px";
    this.loadingStyle.lineHeight = this.coverWidth + "px";

    // for touch event
    window.addEventListener("touchstart", this.onTouchStart);
    window.addEventListener("touchmove", this.onTouchMove);
    window.addEventListener("touchend", this.onTouchEnd);

    // player initiating
    this.cur.src = this.songs[0].url;
  },
  created() {
    let vm = this;
    this.handleScroll = function() {
      setTimeout(function() {
        vm.onScroll();
      }, 200);
    };
    window.addEventListener("scroll", this.handleScroll, {
      passive: true
    });
    this.coverWidth = Math.min(window.innerWidth, 870) * 0.8;
    this.coverHeight = this.coverWidth;
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.handleScroll);
  },
  methods: {
    onScroll() {
      if (this.height - window.scrollY >= 0) {
        this.playerStyle.opacity = (this.height - window.scrollY) / this.height;
      } else {
        this.playerStyle.opacity = 0;
      }
    },
    onNext() {
      this.prog = this.all;
      this.next();
    },
    onPlaylistClick(index) {
      if (this.curIndex === index) return;
      if (index >= 0 && index < this.songs.length) {
        this.curIndex = index;
        this.pause();
        this.cur.currentTime = 0;
        this.cur.src = this.songs[this.curIndex].url;
      }

      this.song = document.querySelector("audio");
      let vm = this;
      let onPlay = function() {
        console.log("good to go");
        vm.play();
        vm.song.removeEventListener("canplay", onPlay);
      };
      this.song.addEventListener("canplay", onPlay);
    },
    play() {
      let vm = this;

      this.duration = this.cur.duration;
      this.currentTime = this.cur.currentTime;

      this.songs[this.curIndex].playing = true;
      this.cur.play();
      this.playing = true;

      this.timer = setInterval(function() {
        vm.currentTime = vm.cur.currentTime;
        // console.log(vm.currentTime);
        vm.$emit("updatetimer", vm.currentTime);
      }, 100);

      this.cur.addEventListener("ended", this.onNext);
    },
    pause() {
      this.playing = false;
      this.songs[this.curIndex].playing = false;
      this.cur.pause();
      // this.cur.pause();
      clearInterval(this.timer);
    },
    next() {
      this.pause();

      // handle currently playing song
      this.songs[this.curIndex].playing = false;
      this.cur.removeEventListener("ended", this.onNext);
      this.cur.currentTime = 0;

      // next song
      this.curIndex++;
      if (this.curIndex > this.songs.length - 1) {
        this.curIndex = 0;
      }

      this.cur.src = this.songs[this.curIndex].url;

      this.song = document.querySelector("audio");
      let vm = this;
      let onPlay = function() {
        console.log("good to go");
        vm.play();
        vm.song.removeEventListener("canplay", onPlay);
      };
      this.song.addEventListener("canplay", onPlay);
    },
    prev() {
      this.pause();

      // handle currently playing song
      this.songs[this.curIndex].playing = false;
      this.cur.removeEventListener("ended", this.onNext);
      this.cur.currentTime = 0;

      // previous song
      this.curIndex--;
      if (this.curIndex < 0) {
        this.curIndex = this.songs.length - 1;
      }

      this.cur.src = this.songs[this.curIndex].url;

      this.song = document.querySelector("audio");
      let vm = this;
      let onPlay = function() {
        console.log("good to go");
        vm.play();
        vm.song.removeEventListener("canplay", onPlay);
      };
      this.song.addEventListener("canplay", onPlay);
    },
    updateProgress(e) {
      // this.pause();
      if (!this.playing) this.play();
      this.cur.currentTime = this.duration * (e.offsetX / this.all);
      this.currentTime = this.cur.currentTime;
      this.prog = this.all * (this.currentTime / this.duration);

      // this.play();
    },
    startDragging() {
      this.dragging = true;
    },
    stopDragging(e) {
      if (this.dragging) {
        this.dragging = false;
        this.updateProgress(e);
      }
    },
    updateProgressDragging(e) {
      if (this.dragging) {
        this.prog =
          this.all * ((this.duration * (e.offsetX / this.all)) / this.duration);
      }
    },
    onTouchStart: function(e) {
      this.startX = e.touches[0].clientX;
      this.startY = e.touches[0].clientY;
      this.x = this.startX;
      this.y = this.startY;
    },
    onTouchMove: function(e) {
      this.x = e.touches[0].clientX;
      this.y = e.touches[0].clientY;
    },
    onTouchEnd: function() {
      let swipeX = this.startX - this.x;
      let swipeY = this.startY - this.y;
      let indicator = Math.abs(swipeY) - Math.abs(swipeX);
      if (indicator > 0) {
        // vertical
        if (swipeY > 0) {
          // swipe up
          if (swipeY > 0.05 * this.windowHeight) {
            this.showPlaylist = true;
          }
        } else {
          // swipe down

          if (swipeY < -0.1 * this.windowHeight || window.scrollY === 0) {
            this.showPlaylist = false;
          }
        }
      } else {
        // horizontal
        if (swipeX > 0) {
          // swipe left
          if (swipeX > 0.1 * this.windowWidth && !this.showPlaylist) {
            this.show = false;
          }
        } else {
          // swipe right
          if (swipeX < -0.1 * this.windowWidth) {
            this.show = true;
          }
        }
      }
    }
  },
  watch: {
    currentTime: function() {
      this.prog = this.all * (this.currentTime / this.duration);
    }
  },
  filters: {
    timeformatter: function(value) {
      let hour = 0;
      let min = 0;
      let sec = 0;
      hour = Math.floor(value / 3600);
      value %= 3600;
      min = Math.floor(value / 60);
      sec = Math.floor(value % 60);
      return hour === 0 ? `${min}:${sec}` : `${hour}:${min}:${sec}`;
    }
  }
};
</script>

<style scoped>
html,
body {
  margin: 0px;
  padding: 0px;
  font-size: 14px;
}

.player-view {
  width: 100%;
  max-width: 870px;
  margin: auto;
}

#player {
  display: block;
  width: 90%;
  margin: 0 auto;
  position: sticky;
  top: 0px;
  /* height: 90vh; */
}

#player header {
  height: 60px;
}

.cover {
  display: block;
  /* width: 80%; */
  box-shadow: 2px 2px 7px rgba(128, 128, 128, 0.456);
  margin: 10% auto;
}

.loading {
  background-color: rgba(240, 248, 255, 0.311);
  position: absolute;
  font-size: 100px;
  text-align: center;
}

span.detail {
  display: block;
  margin-top: 30%;
}

#progress-container {
  background-color: white;
  border: 1px solid wheat;
  height: 3px;
  width: 100%;
}

span.now {
  position: absolute;
  left: 0px;
}

span.all {
  position: absolute;
  right: 0px;
}

.progress {
  background-color: wheat;
  height: 3px;
  border-right: 1px solid rosybrown;
  transition: width 0.1s linear;
}

.button-container {
  width: 90%;
  margin: 40px auto;
  display: flex;
  justify-content: space-between;
}

button {
  height: 60px;
  width: 60px;
  margin: 20px 10px;
  border: none;
  outline: none;
}

button.play {
  background: url("../assets/icons/play.svg") no-repeat center left;
  background-size: 60px 60px;
}

button.stop {
  background: url("../assets/icons/pause.svg") no-repeat center left;
  background-size: 60px 60px;
}

button.next {
  background: url("../assets/icons/next.svg") no-repeat center left;
  background-size: 60px 60px;
}

button.prev {
  background: url("../assets/icons/back.svg") no-repeat center left;
  background-size: 60px 60px;
}

#playlist {
  width: 90%;
  margin: 0px auto;
  position: relative;
  /* height: 100vh; */
  border: 1px wheat solid;
  border-radius: 5px;
  margin-bottom: 10px;
  background-color: rgb(255, 253, 248);
}

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

#playlist ul {
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

#lyrics {
  position: absolute;
  width: 100%;
  left: 0px;
  top: 0;
  overflow-y: hidden;
}

/* player transition */
.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.5s ease;
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active for below version 2.1.8 */ {
  transform: translateX(-400px);
  opacity: 0;
}

/* playlist transition */
.down-slide-fade-enter-active {
  transition: all 0.3s ease;
}
.down-slide-fade-leave-active {
  transition: all 0.3s ease;
}
.down-slide-fade-enter, .down-slide-fade-leave-to
/* .slide-fade-leave-active for below version 2.1.8 */ {
  transform: translateY(100%);
  opacity: 0.7;
}

/* lyrics transition */
.reverse-slide-fade-enter-active {
  transition: all 0.4s ease;
}
.reverse-slide-fade-leave-active {
  transition: all 0.3s ease;
}

.reverse-slide-fade-enter {
  transform: translateX(80%);
  opacity: 0;
}
.reverse-slide-fade-enter-to {
  transform: translateX(0);
}

.reverse-slide-fade-leave {
  transform: translateX(0);
}
.reverse-slide-fade-leave-to {
  transform: translateX(80%);
  opacity: 0;
}
</style>

