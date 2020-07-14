<template>
  <div id="lyrics" ref="lyrics">
    <ul class="lyrics-view-wrapper" ref="lyricsText">
      <li
        class="line-container"
        v-for="(l, i) in lyricArray"
        :key="i"
        :class="{
          highlight: i===curIndex
        }"
        :style="{opacity:Math.max(1-(curIndex-i)*0.3,0)}"
      >
        <span>{{ l }}</span>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: ["timestamp", "currentMusicIndex", "touching", "showing"],
  data() {
    return {
      lyric:
        "[00:00.000] 作曲 : Toru/milet\n[00:01.000] 作词 : milet\n[00:03.64]Where you’re going\n[00:05.90]まだ明けない夜は\n[00:08.06]愛想を尽かして\n[00:10.41]期待はもうしない\n[00:13.38]あなたはもういない\n[00:17.78]凍りついた声\n[00:20.13]誰も溶かすことできずに\n[00:23.47]ただ広がるspace\n[00:26.07]青いままのflames\n[00:28.62]遠く消える テールライト 照らして\n[00:34.91]二つになった 影を残して\n[00:43.85]Tell me what is inside you inside you now\n[00:50.20]手に触れた 最初で最後でも\n[00:56.47]Tell me who is inside 許されるなら\n[01:03.74]Let me in you again\n[01:06.28]and just stay Please just stay\n[01:10.99]エンドロールは流れない時は\n[01:15.36]もう何も癒さない\n[01:17.62]繰り返しのscene\n[01:20.25]瞳の裏貼りついたscreen\n[01:22.91]Don’t say\n[01:23.98]it’s too late to say I need you\n[01:29.24]Don’t say\n[01:30.27]it’s too late to say I miss you\n[01:38.63]Tell me what is inside you inside you now\n[01:44.90]目に見えない 最初が最後でも\n[01:51.33]Tell me who is inside 許されるなら\n[01:58.47]Let me in you again\n[02:01.20]今だけ そばにいて\n[02:12.21]Maybe you’re right\n[02:13.68]Maybe your life’s better without me\n[02:18.72]でも知らない 誰も知らない\n[02:21.72]あなたがいたってoh\n[02:24.84]何度も確かめるほどに\n[02:28.59]砂のようにこぼれ落ちた\n[02:32.82]もうきっと 戻せないの\n[02:39.35]Tell me what is inside you inside you now\n[02:45.63]手に触れた 最初で最後でも\n[02:52.19]Tell me who is inside 許されるなら\n[02:59.33]Let me in you again\n[03:02.30]and just stay Please just stay\n[03:06.64]Maybe you’re right\n[03:08.17]Maybe your life’s better without me\n[03:12.96]I need you here so\n[03:14.53]I need to hear you more\n[03:16.29]never let you down\n[03:19.50]Maybe you’re right\n[03:20.97]Maybe your life’s better without me\n[03:25.82]Maybe You’re right\n[03:27.15]But let me in your life\n[03:28.93]just one night\n",
      lyricArray: [],
      timer: 0,
      timeArray: [],
      positionTop: 0,
      prevPosition: 0,
      prevIndex: 0,
      off: 0,
      chosenTimestamp: 0,
      onelineHeight: 0,
      curIndex: 0,
      scrollDis: 0,
      timeout: null,
      offset: 0
    };
  },
  mounted() {
    this.makeArray();
    this.timer = this.timestamp;

    for (let i = 0; i < this.timeArray.length; i++) {
      if (
        this.timer >= this.timeArray[i] &&
        this.timer < this.timeArray[i + 1]
      ) {
        this.curIndex = i;
        break;
      }
    }
    let vm = this;
    setTimeout(function() {
      // console.log(document.querySelector("#lyrics").scrollHeight);
      // console.log(vm.curIndex);
      window.scrollTo(0, vm.curIndex * 48);
    }, 300);

    // let a = document.querySelector(".lyrics-view-wrapper");
  },
  methods: {
    makeArray() {
      let arr = this.lyric.split("\n");
      // console.log("arr", arr);
      this.timeArray.push(0);
      this.lyricArray.push("");

      // console.log(start, end);
      for (let i of arr) {
        let l = i.split("]");
        let time = l[0];
        let line = l[1];

        time = time.slice(1).split(":");
        let m = Number(time[0]);
        let s = Number(time[1]);
        let timestamp = m * 60 + s;

        if (timestamp !== 0 && !timestamp) {
          this.timeArray.push(999);
          this.lyricArray.push("");
        } else {
          this.timeArray.push(timestamp);
          this.lyricArray.push(line);
        }
      }
      // console.log(this.lyricArray, this.timeArray);
    },
    onScroll() {
      // e.preventDefault();

      let p = this.prevPosition;
      let vm = this;
      this.offset = window.scrollY - p;
      if (this.timeout != null) {
        window.clearTimeout(this.timeout);
      }
      this.timeout = window.setTimeout(() => {
        // if (vm.) {
        //   window.removeEventListener("scroll", vm.onScroll);
        // }
        if (vm.offset !== 0) {
          // console.log("stopped");
          window.removeEventListener("scroll", vm.onScroll);

          window.scrollTo(0, vm.curIndex * 48);
          vm.chosenTimestamp = vm.timeArray[vm.curIndex];
          vm.$emit("updateTimestampByLyrics", vm.chosenTimestamp);
          // console.log(vm.chosenTimestamp);
        }
      }, 200);
    }
  },
  watch: {
    timestamp: function() {
      // console.log(this.timestamp);
      this.timer = this.timestamp;
      // console.log(this.timer);
      if (
        this.timer + 0.2 > this.timeArray[this.curIndex + 1] &&
        this.timer + 0.2 < this.timeArray[this.curIndex + 2]
      ) {
        // go to next line
        this.curIndex++;
        // console.log(this.curIndex);
        window.scrollBy({
          top: 48,
          left: 0,
          behavior: "smooth"
        });
      }
    },
    currentMusicIndex: function() {
      // change song
      this.curIndex = 0;
      window.scrollTo(0, 0);
    },
    offset: function() {
      this.off = Math.round(this.offset / 48);
      if (this.prevIndex + this.off < 0) {
        this.curIndex = 0;
      } else if (this.prevIndex + this.off > this.timeArray.length - 1) {
        this.curIndex = this.timeArray.length - 1;
      } else {
        this.curIndex = this.prevIndex + this.off;
      }
    },
    touching: function() {
      if (this.touching) {
        // console.log("touching");
        this.prevPosition = window.scrollY;
        this.prevIndex = this.curIndex;
        window.addEventListener("scroll", this.onScroll);
      } else {
        // console.log("stoptouching");
      }
    }
  }
};
</script>

<style scoped>
#lyrics {
  width: 100%;
  overflow: hidden;
  /* height: 80%; */
}

.lyrics-view-wrapper {
  text-align: center;
  padding: 100px 0px 110%;
  margin: 0px;
  height: 100%;
  margin: auto;
  width: 100%;
}

.line-container {
  display: inline-block;
  width: 100%;
  height: 3rem;
  line-height: 3rem;
}
span {
  width: 80%;
  margin: auto;
  padding: 0px;
  display: inline-block;
  line-height: 1rem;
  transition: color 0.2s linear;
}

.highlight {
  color: white;
  font-size: 1.3rem;
  height: 6rem;
  line-height: 6rem;
}

.highlight span {
  line-height: 1.5rem;
}
</style>
