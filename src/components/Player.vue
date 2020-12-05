<template>
  <div class="player-view" ref="view">
    <div
      id="player"
      @mousemove="updateProgressDragging"
      @mouseup="stopDragging"
      :class="{ maxWidth: isPC }"
    >
      <!-- <header></header> -->
      <audio :src="this.songs[this.curIndex].url"></audio>
      <div
        class="cover"
        :style="{
          background: `url(${require('../assets/covers/' +
            songs[curIndex].cover)})`,
        }"
        :class="{ blur: show }"
      >
        <div
          class="progress-bar"
          @mousedown.stop="startDragging"
          @click="updateProgress"
        >
          <div id="progress-container">
            <div
              class="progress"
              :style="{ width: prog + 'px' }"
              ref="progress"
            ></div>
          </div>
        </div>
      </div>
      <!-- <img
        :src="require('../assets/covers/' + this.songs[this.curIndex].cover)"
        alt="cover"
        class="cover"
        :class="{ blur: show }"
        ref="coverImg"
      /> -->
      <div class="control-container" v-show="!show">
        <span class="now">{{ currentTime | timeformatter }}</span>
        <span class="all">{{ duration | timeformatter }}</span>
        <div class="detail-container">
          <p
            class="detail"
            style="font-weight: bold; margin-bottom: 22px; margin-top: 20px"
          >
            {{ songs[curIndex].title }}
          </p>
          <p class="detail" style="color: #666; font-size: 1.1rem">
            {{ songs[curIndex].author }}
          </p>
        </div>

        <div class="button-container">
          <button class="round prev" @click="prev"></button>
          <button class="play" v-if="!playing" v-on:click="play"></button>
          <button class="stop" v-else @click="pause"></button>
          <button class="round next" @click="next"></button>
        </div>
      </div>
    </div>
    <Option
      id="option"
      @clickPlaylist="onPlaylistClick"
      v-if="showPlaylist"
      v-bind:songList="songs"
      v-bind:current="curIndex"
      :touching="touching"
    ></Option>
    <transition name="slide">
      <lyrics
        id="lyrics"
        v-if="show || isPC"
        v-bind:timestamp="currentTime"
        :currentMusicIndex="curIndex"
        :touching="touching"
        @updateTimestampByLyrics="updateTimestamp"
        :showing="show"
        :class="{ alignRight: isPC }"
      ></lyrics>
    </transition>
  </div>
</template>

<script>
import Option from "./Option.vue";
import Lyrics from "./Lyrics";

export default {
  components: {
    Option,
    Lyrics,
  },
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
          author: "milet",
          title: "Inside You",
          cover: "cover.jpg",
          playing: false,
          url: require("../assets/songs/inside.mp3"),
        },
        {
          song: new Audio(),
          author: "花澤香菜 (はなざわ かな)",
          title: "恋愛サーキュレーション (恋爱循环) (《化物语》TV动画片头曲) ",
          cover: "cover3.jpg",
          playing: false,
          url: require("../assets/songs/123.mp3"),
        },
      ],
      curIndex: 0,
      playing: false,
      isScrolled: false,
      dragging: false,
      coverHeight: 0,
      coverWidth: 0,
      song: null,
      loading: true,
      loadingStyle: {
        height: 0,
        width: 0,
        top: 0,
        left: 30 + "px",
      },
      windowHeight: 0,
      windowWidth: 0,
      isPC: false,
      // touch event
      startX: 0,
      startY: 0,
      x: 0,
      y: 0,
      showPlaylist: false,
      show: false,
      offset: 0,
      touching: false,

      backgroundImg: "",
    };
  },
  mounted() {
    this.windowHeight = window.innerHeight;
    this.windowWidth = window.innerWidth;
    if (this.windowWidth > 800) {
      this.isPC = true;
    }
    this.height = this.windowHeight / 1.5;

    // the loading filter
    // let cover = document.querySelector("img.cover");
    this.loadingStyle.width = this.coverWidth + "px";
    this.loadingStyle.height = this.coverWidth + "px";
    // this.loadingStyle.left = cover.offsetLeft + "px";
    // this.loadingStyle.top = cover.offsetTop + "px";
    this.loadingStyle.lineHeight = this.coverWidth + "px";

    // player initiating
    this.cur.src = this.songs[0].url;
    setTimeout(() => {
      this.getProgressBarWidth();
    });
  },
  created() {
    this.coverWidth = Math.min(window.innerWidth, 570) * 0.7;
    this.coverHeight = this.coverWidth;

    // if (window.innerWidth >= 1140) {
    //   this.show = true;
    // }

    // for touch event
    window.addEventListener("touchstart", this.onTouchStart, {
      passive: false,
    });
    window.addEventListener("touchmove", this.onTouchMove, { passive: false });
    window.addEventListener("touchend", this.onTouchEnd);
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.handleScroll);
  },
  methods: {
    onNext() {
      this.prog = this.all;
      this.next();
    },
    getProgressBarWidth() {
      this.all = document.getElementById("progress-container").clientWidth;
      console.log(this.all);
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
      let onPlay = function () {
        console.log("good to go");
        vm.play();
        vm.song.removeEventListener("canplay", onPlay);
      };
      this.song.addEventListener("canplay", onPlay);
    },
    togglePlaylist() {
      this.showPlaylist = !this.showPlaylist;
      let vm = this;
      let closePlaylist = function () {
        vm.showPlaylist = false;
        window.removeEventListener("click", closePlaylist);
      };
      window.addEventListener("click", closePlaylist);
    },
    play() {
      let vm = this;
      console.log("play");

      this.duration = this.cur.duration;
      this.currentTime = this.cur.currentTime;

      this.songs[this.curIndex].playing = true;
      this.cur.play();
      this.playing = true;

      this.timer = setInterval(function () {
        vm.currentTime = vm.cur.currentTime;
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
      let onPlay = function () {
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
      let onPlay = function () {
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
      let vm = this;
      this.timer = setInterval(function () {
        vm.currentTime = vm.cur.currentTime;
      }, 100);

      // this.play();
    },
    startDragging() {
      this.dragging = true;
      clearInterval(this.timer);
    },
    stopDragging(e) {
      console.log(e);
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
    onTouchStart: function (e) {
      this.startX = e.touches[0].clientX;
      this.startY = e.touches[0].clientY;
      this.x = this.startX;
      this.y = this.startY;
      if (this.show) {
        this.touching = true;
      }
      // console.log("start");
    },
    onTouchMove: function (e) {
      this.x = e.touches[0].clientX;
      this.y = e.touches[0].clientY;
      if (this.show === true) {
        // record the distance that cursor travelled
        // console.log(e);
        this.offset = this.y - this.startY;
      }
      // console.log("moving");
    },
    onTouchEnd: function () {
      // console.log("up");
      let swipeX = this.startX - this.x;
      let swipeY = this.startY - this.y;
      let indicator = Math.abs(swipeY) - Math.abs(swipeX);
      this.touching = false;
      if (indicator > 0) {
        // vertical
        if (swipeY > 0) {
          // swipe up
          if (swipeY > 0.05 * this.windowHeight && this.show === true) {
            // this.showPlaylist = true;
          }
        } else {
          // swipe down
          if (swipeY < -0.1 * this.windowHeight && this.show === true) {
            // this.showPlaylist = false;
          }
        }
      } else {
        // horizontal
        if (swipeX > 0) {
          // swipe left
          // console.log("swipe left!");
          if (swipeX > 0.1 * this.windowWidth && !this.showPlaylist) {
            console.log("swipe left");
            this.show = true;
          }
        } else {
          // swipe right
          // console.log("swipe right!");
          if (swipeX < -0.1 * this.windowWidth) {
            console.log("swipe right");
            this.show = false;
          }
        }
      }
    },
    updateTimestamp(val) {
      this.cur.currentTime = val;
      this.currentTime = val;
      // console.log(val);
    },
  },
  watch: {
    currentTime: function () {
      this.prog = this.all * (this.currentTime / this.duration);
    },
    curIndex: function () {
      this.$emit("updateCurIndex", this.songs[this.curIndex].cover);
    },
  },
  computed: {
    playerleft: function () {
      return this.windowWidth * 0.05 + "px";
    },
  },
  filters: {
    timeformatter: function (value) {
      let hour = 0;
      let min = 0;
      let sec = 0;
      hour = Math.floor(value / 3600);
      value %= 3600;
      min = Math.floor(value / 60);
      sec = Math.floor(value % 60);
      return hour === 0 ? `${min}:${sec}` : `${hour}:${min}:${sec}`;
    },
  },
};
</script>

<style scoped>
* {
  margin: 0px;
  padding: 0px;
}
.player-view {
  width: 100vw;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

#player {
  height: 100%;
  display: block;
  width: 100vw;
  position: fixed;
  top: 0px;
  /* left: 5px; */
  background-color: white;
  overflow: hidden;
  /* max-width: 500px; */
}

.maxWidth {
  max-width: 40%;
}

#player header {
  height: 20%;
}

.cover {
  display: block;
  transition: all 0.3s ease-in-out;
  border: none;
  margin: auto;
  width: 100%;
  padding-bottom: 100%;
  background-size: cover !important;
  position: relative;
}

.blur {
  /* height: 100% !important;
  width: auto !important; */
  /* height: 100%; */
  /* position: fixeßd; */
  transform: scale(3) translateY(20%);
  filter: blur(2rem);
}

.loading {
  background-color: rgba(229, 229, 229, 0.311);
  position: absolute;
  font-size: 100px;
  text-align: center;
}

.control-container {
  /* position: absolute; */
  width: 100%;
  /* bottom: 0px; */
  /* padding-top: 20%; */
}

.control-container img {
  left: 0px;
  height: 100%;
  display: inline-block;
}

.detail-container {
  display: inline-block;
  margin-top: 30px;
}
.detail {
  display: block;
  line-height: 1.2rem;
  height: 1.2rem;
  white-space: nowrap;
  font-size: 1.5rem;
  font-weight: 400;
}

button.show-lyrics {
  display: inline-block;
  width: 30px;
  height: 30px;
}

.progress-bar {
  height: 100%;
  width: 100%;
}

#progress-container {
  background-color: #ececec17;
  /* height: 6px; */
  width: 100%;
  margin: auto;
  border: none;
  box-shadow: 0px 1px 7px 0px rgba(98, 98, 98, 0.34);
  /* position: relative; */
  height: 100%;
  top: 0px;
  position: absolute;
  /* position: fixed; */
  /* max-width: 500px; */
  /* max-height: 500px; */
}

span.now {
  position: absolute;
  left: 1rem;
}

span.all {
  position: absolute;
  right: 1rem;
}

.progress {
  background-color: rgb(255 255 255 / 30%);
  /* height: 6px; */
  position: absolute;
  /* border-right: 1px solid rgb(196, 196, 196); */
  top: 0px;
  height: 100%;
}

.button-container {
  width: 80%;
  margin: 10% auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

button {
  height: 30px;
  width: 30px;
  /* margin: 20px 10px; */
  border: none;
  outline: none;
}

button.play {
  background: url("../assets/icons/play.svg") no-repeat center left;
  background-size: 100%;
  height: 60px;
  width: 60px;
}

button.stop {
  background: url("../assets/icons/pause.svg") no-repeat center left;
  background-size: 100%;
  height: 60px;
  width: 60px;
}

button.next {
  background: url("../assets/icons/next.svg") no-repeat center left;
  background-size: 100%;
}

button.prev {
  background: url("../assets/icons/back.svg") no-repeat center left;
  background-size: 100%;
}

#lyrics {
  position: absolute;
  width: 100%;
  left: 0px;
  top: 0px;
  overflow-y: hidden;
}

#lyrics.alignRight {
  width: 60%;
  right: 0px;
  left: unset;
}

/* Lyrics transition */
.slide-enter,
.slide-leave-to {
  transform: translateX(50%);
  opacity: 0;
}
.slide-enter-active {
  transition: all 0.2s ease;
}
.slide-leave-active {
  transition: all 0.2s cubic-bezier(1, 0.5, 0.8, 1);
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

#option {
  width: 100%;
  margin: 0px auto;
  position: absolute;
  /* height: 100vh; */
  /* border: 1px wheat solid; */
  border-radius: 5px;
  /* margin-bottom: 10px; */
  background-color: rgb(255, 253, 248);
  z-index: 0;
  bottom: 0px;
}
</style>

