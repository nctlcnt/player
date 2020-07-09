<template>
  <div id="lyrics" ref="lyrics">
    <ul ref="lyricsText">
      <div
        class="line-container"
        v-for="(l, i) in lyricArray"
        :key="i"
        :class="{
          highlight: i===curIndex
        }"
      >
        <span>{{ l | formatter }}</span>
      </div>
    </ul>
  </div>
</template>

<script>
export default {
  props: ["timestamp", "currentMusicIndex", "offset", "touching"],
  data() {
    return {
      lyric:
        "[00:00.56]セーノ\n[00:01.41]でも そんなんじゃ だめでも そんなんじゃ だめ\n[00:03.22]もう そんなんじゃ ほら\n[00:05.24]心は進化するよ\n[00:07.51]もっともっと\n[00:09.60]恋愛サーキュレーション\n[00:25.32]言葉にすれば消えちゃう関係なら\n[00:27.73]言葉を消せばいいやって\n[00:29.55]思ってた 恐れてた\n[00:31.18]だけど あれ？なんか違うかも\n[00:33.28]せんりのみちもいっぽから！\n[00:35.22]石のようにかたい そんな意志で\n[00:37.50]ちりもつもればやまとなでしこ？\n[00:39.53]「し」抜きで いや 死ぬ気で！\n[00:41.54]ふわふわり ふわふわる\n[00:44.50]あなたが名前を呼ぶ\n[00:46.55]それだけで\n[00:47.51]宙へ浮かぶ\n[00:49.54]ふわふわる ふわふわり\n[00:52.52]あなたが笑っている\n[00:54.57]それだけで\n[00:55.58]笑顔になる\n[00:57.53]神様 ありがとう\n[01:01.27]運命のいたずらでも\n[01:05.52]めぐり逢えたことが\n[01:09.27]しあわせなの\n[01:13.29]でも そんなんじゃ だめ\n[01:15.32]もう そんなんじゃ ほら\n[01:17.27]心は進化するよ\n[01:19.49]もっと もっと\n[01:21.29]そう そんなんじゃ やだ\n[01:23.31]ねぇ そんなんじゃ まだ\n[01:25.26]私のこと 見ててね\n[01:27.51]ずっと ずっと\n[01:29.56]...\n[01:45.22]私の中のあなたほど\n[01:47.00]あなたの中の私の存在は\n[01:49.48]まだまだ 大きくないことも\n[01:52.06]わかっているけれど\n[01:53.37]今この同じ 瞬間\n[01:55.28]共有している 実感\n[01:57.36]ちりもつもればやまとなでしこ！\n[01:59.49]略してちりつもやまとなでこ！\n[02:01.54]くらくらり くらくらる\n[02:04.51]あなたを見上げたら\n[02:06.57]それだけで\n[02:07.52]まぶしすぎて\n[02:09.52]くらくらる くらくらり\n[02:12.52]あなたを思っている\n[02:14.44]それだけで\n[02:15.50]とけてしまう\n[02:17.53]神様 ありがとう\n[02:21.29]運命のいたずらでも\n[02:25.47]めぐり逢えたことが\n[02:29.22]しあわせなの\n[02:33.48]コイスル キセツハ ヨクバリ サーキュレーション\n[02:41.53]コイスル キモチハ ヨクバリ サーキュレーション\n[02:49.52]コイスル ヒトミハ ヨクバリ サーキュレーション\n[02:57.52]コイスル オトメハ ヨクバリ サーキュレーション\n[03:07.58]ふわふわり ふわふわる\n[03:10.47]あなたが名前を呼ぶ\n[03:12.57]それだけで\n[03:13.49]宙へ浮かぶ\n[03:15.48]ふわふわる ふわふわり\n[03:18.47]あなたが笑っている\n[03:20.45]それだけで\n[03:21.47]笑顔になる\n[03:23.52]神様 ありがとう\n[03:27.24]運命のいたずらでも\n[03:31.57]めぐり逢えたことが\n[03:35.26]しあわせなの\n[03:39.53]...\n[03:55.28]でも そんなんじゃ だめ\n[03:57.30]もう そんなんじゃ ほら\n[03:59.27]心は進化するよ\n[04:01.58]もっと もっと\n[04:03.26]そう そんなんじゃ やだ\n[04:05.22]ねぇ そんなんじゃ まだ\n[04:07.25]私のこと 見ててね\n[04:09.49]ずっと ずっと\n",
      lyricArray: [],
      timer: 0,
      timeArray: [],
      curIndex: 0,
      allScroll: 0,
      positionTop: 0,
      prevPosition: 0,
      prevIndex: 0,
      off: 0,
      chosenTimestamp: 0,
      onelineHeight: 0
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

    this.positionTop = -this.curIndex * 48;

    let a = document.querySelector("ul");
    a.style.top = this.positionTop + "px";

    this.allScroll = document.querySelector("#lyrics").scrollHeight;
    console.log(this.allScroll);

    // this.windowHeight = window.innerHeight;
    // document.querySelector("#lyrics").style.height = this.windowHeight + "px";
  },
  methods: {
    makeArray() {
      let arr = this.lyric.split("\n");
      this.timeArray.push(0);
      this.lyricArray.push("");
      for (let i of arr) {
        let time = i.slice(1, 8).split(":");
        let m = Number(time[0]);
        let s = Number(time[1]);
        let timestamp = m * 60 + s;

        if (!timestamp) {
          this.timeArray.push(999);
          this.lyricArray.push("");
        } else {
          this.timeArray.push(timestamp);
          this.lyricArray.push(i.slice(8));
        }
      }
    },
    highlightLine(i) {
      let line = this.lyricArray[i];
      // console.log(line);
      if (!line) {
        return 0;
      } else {
        return this.getTimestamp(line);
      }
    }
  },
  watch: {
    timestamp: function() {
      this.timer = this.timestamp;
      if (
        this.timer > this.timeArray[this.curIndex + 1] &&
        this.timer < this.timeArray[this.curIndex + 2]
      ) {
        // go to next line
        this.curIndex++;
        let a = document.querySelector("ul");

        console.log(this.onelineHeight);
        this.positionTop -= this.onelineHeight;
        a.style.top = this.positionTop + "px";
      }
    },
    curIndex: function() {
      let a = document.querySelector("ul");
      console.log("ul", a.offsetTop);
      this.onelineHeight = 48;
      console.log("one", this.onelineHeight);
    },
    currentMusicIndex: function() {
      // change song
      this.curIndex = 0;
      this.positionTop = 0;
    },
    offset: function() {
      // handle the touch-drag event
      // console.log(this.offset);
      this.off = -Math.round(this.offset / 48);
      // console.log(this.curIndex, "=", this.prevIndex, "+", this.off);
      if (this.prevIndex + this.off < 0) {
        this.curIndex = 0;
      } else if (this.prevIndex + this.off > this.timeArray.length - 1) {
        this.curIndex = this.timeArray.length - 1;
      } else {
        this.curIndex = this.prevIndex + this.off;
      }
      let a = document.querySelector("ul");
      this.positionTop = this.prevPosition + this.offset;

      if (this.positionTop >= 0) {
        this.positionTop = 0;
      }
      a.style.top = this.positionTop + "px";
    },
    touching: function() {
      if (this.touching) {
        console.log("touching");
        this.prevPosition = this.positionTop;
        this.prevIndex = this.curIndex;
      } else {
        console.log("stoptouching");
        this.chosenTimestamp = this.timeArray[this.curIndex];
        this.$emit("updateTimestampByLyrics", this.chosenTimestamp);

        this.positionTop = this.prevPosition - this.off * 48;
        console.log(
          "top",
          this.positionTop,
          "=",
          this.prevPosition,
          "+",
          this.offset
        );
        if (this.positionTop >= 0) {
          this.positionTop = 0;
        }
        let a = document.querySelector("ul");
        a.style.top = this.positionTop + "px";
      }
    }
  },
  filters: {
    formatter(l) {
      return l.split("]")[1];
    }
  }
};
</script>

<style scoped>
#lyrics {
  width: 100%;
  overflow: hidden;
  height: 100%;
}

ul {
  text-align: center;
  padding: 70% 0px;
  margin: 0px;
  position: absolute;
  /* top: 0px; */
  height: 100%;
  margin: auto;
  width: 100%;
}
.line-container {
  display: inline-block;
  width: 100%;
  height: 3rem;
}
span {
  width: 80%;
  margin: auto;
  padding: 0px;
  display: inline-block;
  line-height: 1rem;
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
