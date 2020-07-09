<template>
  <div class="player-view" ref="view">
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
        @click.stop="togglePlaylist"
        :class="{blur:show}"
      />
      <div class="control-container" v-show="!show">
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
        v-if="show"
        v-bind:timestamp="currentTime"
        :currentMusicIndex="curIndex"
        :offset="offset"
        :touching="touching"
        @updateTimestampByLyrics="updateTimestamp"
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
    Lyrics
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
      show: false,
      offset: 0,
      touching: false
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

    // player initiating
    this.cur.src = this.songs[0].url;
  },
  created() {
    this.coverWidth = window.innerWidth * 0.8;
    this.coverHeight = this.coverWidth;

    // for touch event
    window.addEventListener("touchstart", this.onTouchStart, {
      passive: false
    });
    window.addEventListener("touchmove", this.onTouchMove, { passive: false });
    window.addEventListener("touchend", this.onTouchEnd);
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
    togglePlaylist() {
      this.showPlaylist = !this.showPlaylist;
      let vm = this;
      let closePlaylist = function() {
        vm.showPlaylist = false;
        window.removeEventListener("click", closePlaylist);
      };
      window.addEventListener("click", closePlaylist);
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
      let vm = this;
      this.timer = setInterval(function() {
        vm.currentTime = vm.cur.currentTime;
      }, 100);

      // this.play();
    },
    startDragging() {
      this.dragging = true;
      clearInterval(this.timer);
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
      this.touching = true;
    },
    onTouchMove: function(e) {
      this.x = e.touches[0].clientX;
      this.y = e.touches[0].clientY;
      if (this.show === true) {
        // record the distance that cursor travelled
        // console.log(e);
        this.offset = this.y - this.startY;
      }
    },
    onTouchEnd: function() {
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
            this.show = true;
          }
        } else {
          // swipe right
          // console.log("swipe right!");
          if (swipeX < -0.1 * this.windowWidth) {
            this.show = false;
          }
        }
      }
    },
    updateTimestamp(val) {
      this.cur.currentTime = val;
      this.currentTime = val;
      console.log(val);
    }
  },
  watch: {
    currentTime: function() {
      this.prog = this.all * (this.currentTime / this.duration);
    },
    curIndex: function() {
      this.$emit("updateCurIndex", this.songs[this.curIndex].cover);
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
.player-view {
  width: 100%;
  margin: 0;
  padding: 0;
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
  transition: all 0.2s ease-in-out;
}

.blur {
  filter: blur(3rem);
  position: fixed;
  transform: scale(3);
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
  margin: auto;
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

#lyrics {
  position: absolute;
  width: 100%;
  left: 0px;
  top: 0px;
  overflow-y: hidden;
}

/* Lyrics transition */
.slide-enter,
.slide-leave-to {
  transform: translateX(50%);
  opacity: 0;
}
.slide-enter-active {
  transition: all 0.3s ease;
}
.slide-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
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
  border: 1px wheat solid;
  border-radius: 5px;
  margin-bottom: 10px;
  background-color: rgb(255, 253, 248);
  z-index: 0;
  bottom: 0px;
}
</style>

