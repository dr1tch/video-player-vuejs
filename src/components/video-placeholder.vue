<template>
  <div class="player-container">
    <div
      class="loading-bar"
      :style="{ width: `${loadPercent}%` }"
      v-if="loading"
    />
    <video
      :src="src"
      :controls="false"
      class="placeholder"
      :loop="true"
      autoplay
      muted
      playsinline
      ref="video"
      v-show="showVideo"
      v-if="ifVideo"
      @loadstart="loadVideo"
      @progress="atProgress"
    />
    <img class="placeholder" :src="poster" v-show="showPoster" />
  </div>
</template>

<script>
export default {
  name: "player-placeholder",
  props: {
    src: {
      required: false,
      type: String,
    },
    poster: {
      required: false,
      type: String,
    },
    previewOnMouse: {
      type: Boolean,
      default: false,
    },
    mouseover: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      loading: false,
      loadPercent: 0,
    }
  },
  methods: {
    atProgress(e) {
      let range = 0
      let buffered = this.$refs.video.buffered
      let time = this.$refs.video.currentTime

      while (!(buffered.start(range) <= time && time <= buffered.end(range))) {
        range += 1
      }

      let loadStartPercentage =
        buffered.start(range) / this.$refs.video.duration
      let loadEndPercentage = buffered.end(range) / this.$refs.video.duration

      this.loadPercent = (loadEndPercentage - loadStartPercentage) * 100
      if (this.loadPercent >= 100) this.loading = false
    },
    loadVideo(e) {
      this.loading = true
    },
    computed: {
      ifVideo() {
        let result = false
        if (this.previewOnMouse) {
          if (this.mouseover) result = true
          if (this.loadPercent) result = true
        } else {
          result = true
        }
        return result
      },
      showVideo() {
        let result = false
        if (this.previewOnMouse) {
          if (this.src && this.mouseover) result = true
        }
        if (!this.previewOnMouse) {
          if (this.src) result = true
        }
      },
      showPoster() {
        if (this.poster && !this.showVideo) return true
        return false
      },
    },
  },
}
</script>

<style scoped>
.loading-bar {
  position: absolute;
  top: 0;
  left: 0;
  background-color: rgb(192, 192, 192);
  height: 3px;
  z-index: 10;
}

div.player-container {
  width: 100%;
  height: 100%;
}

div.player-container,
img,
video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
