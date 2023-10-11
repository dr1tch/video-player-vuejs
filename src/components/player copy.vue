<template>
  <div
    class="vue-video-player"
    :class="{
      'volume-showing': showSound,
    }"
    @mouseenter="atMouseenter"
    @mouseleave="atMouseleave"
  >
    <video
      ref="video"
      :src="srcComputed"
      :controls="false"
      :autoplay="autoplay"
      :playsinline="playsinline"
      :class="{
        transparent: !started,
      }"
      @play="play"
      @pause="pause"
      @click="atPlayPause"
      @timeupdate="atTimeupdate"
      @canplay="atCanplay"
      @volumechange="atVolumechange"
      @ended="atEnded"
      @stalled="test"
    />

    <PlayerVideoPlaceholder
      :src="videoPlaceholderSrc"
      :poster="poster"
      :preview-on-mouse="previewOnMouse"
      :mouseover="mouseover"
      v-if="!started"
    />

    <div :class="overlayClass" v-if="!started">
      <div class="title" v-if="title">
        {{ title }}
      </div>

      <PlayerButton class="start-button" @click="atPlayPause">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          class="lucide lucide-play"
        >
          <polygon points="5 3 19 12 5 21 5 3" />
        </svg>
      </PlayerButton>
    </div>
    <svg
      v-if="started && loading"
      xmlns="http://www.w3.org/2000/svg"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      stroke-width="2"
      stroke-linecap="round"
      stroke-linejoin="round"
      class="lucide lucide-rotate-ccw"
    >
      <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" />
      <path d="M3 3v5h5" />
    </svg>

    <!-- Use this slot to replace the controls  -->
    <slot name="controls">
      <PlayerControls
        :class="controlsClass"
        :show="showControlsIf"
        :style="controlsStyleComputed"
      >
        <PlayerButton
          :class="playButtonClass"
          :style="playButtonStyle"
          @click="atPlayPause"
          class="start-button"
        >
          <svg
            v-if="!playingComputed"
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="lucide lucide-play"
          >
            <polygon points="5 3 19 12 5 21 5 3" />
          </svg>
          <svg
            v-else
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="lucide lucide-pause"
          >
            <rect width="4" height="16" x="6" y="4" />
            <rect width="4" height="16" x="14" y="4" />
          </svg>
        </PlayerButton>

        <PlayerButton
          @click="atVolume"
          :class="volumeButtonClass"
          :style="volumeButtonStyle"
          class="start-button"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="lucide lucide-volume-2"
          >
            <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5" />
            <path d="M15.54 8.46a5 5 0 0 1 0 7.07" />
            <path d="M19.07 4.93a10 10 0 0 1 0 14.14" />
          </svg>
        </PlayerButton>

        <PlayerRange
          v-model="volumeComputed"
          :max="1"
          width="60px"
          :show="showSound"
          class="start-button"
        />

        <PlayerButton
          @click="atVolume"
          :class="timerClass"
          :style="timerStyle"
          class="start-button"
        >
          {{ minutesTime }} / {{ minutesDuration }}
        </PlayerButton>

        <PlayerButton
          :class="fullscreenButtonClass"
          :style="fullscreenButtonStyle"
          class="start-button"
          @click="atFullscreen"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="lucide lucide-expand"
          >
            <path d="m21 21-6-6m6 6v-4.8m0 4.8h-4.8" />
            <path d="M3 16.2V21m0 0h4.8M3 21l6-6" />
            <path d="M21 7.8V3m0 0h-4.8M21 3l-6 6" />
            <path d="M3 7.8V3m0 0h4.8M3 3l6 6" />
          </svg>
        </PlayerButton>

        <PlayerRange v-model="timeComputed" :max="duration" show />
      </PlayerControls>
    </slot>
  </div>
</template>

<script>
import PlayerControls from "./controls.vue"
import PlayerButton from "./button.vue"
import PlayerRange from "./range.vue"
import PlayerVideoPlaceholder from "./video-placeholder.vue"
import { Play, Expand, Volume2 } from "lucide-vue-next"
export default {
  name: "video-player",
  components: {
    PlayerControls,
    PlayerButton,
    PlayerRange,
    PlayerVideoPlaceholder,
  },
  props: {
    /**
     * title: it appears on the overlays
     */
    title: {
      type: String,
      default: null,
    },
    /**
     * play/pause: it automatically set the video the right state.
     */
    value: {
      type: Boolean,
      default: undefined,
    },
    /**
     * Define the source for the video tag.
     * if array uses the src-index to pick one
     */
    src: {
      required: true,
      type: [String, Array],
    },
    /**
     * Use ONLY if src is a array
     * set the index of the src
     */
    srcIndex: [String, Number],
    /**
     * set the video to autoplay as it's loaded
     */
    autoplay: {
      type: Boolean,
      default: false,
    },
    /**
     * show video preview if mouse hover
     */
    previewOnMouse: {
      type: Boolean,
      default: false,
    },
    /**
     * set the video to playsinline as it's loaded
     */
    playsinline: {
      type: Boolean,
      default: false,
    },
    /**
     * set a image placeholder util the video start to play once
     */
    poster: String,
    /**
     * if you want a video teaser you can use this, will be played with-out sound
     */
    videoPlaceholderSrc: String,
    /**
     * define the volume of the player
     * values from 0 util 1
     */
    volume: {
      type: Number,
      default: undefined,
    },
    /**
     * show/hide the controls
     */
    controls: {
      type: Boolean,
      default: undefined,
    },
    /**
     * class for the overlay
     */
    overlayClass: {
      default: "vue-video-center",
    },
    /**
     * class for the controls bar
     */
    controlsClass: {
      default: "vue-video-player-controls",
    },
    /**
     * style for the controls bar
     */
    controlsStyle: {
      default: null,
    },
    /**
     * class for the play button
     */
    playButtonClass: {
      default: null,
    },
    /**
     * style for the play button
     */
    playButtonStyle: {
      default: null,
    },
    /**
     * class for the volume button
     */
    volumeButtonClass: {
      default: null,
    },
    /**
     * style for the volume button
     */
    volumeButtonStyle: {
      default: null,
    },
    /**
     * class for the timer
     */
    timerClass: {
      default: "timer",
    },
    /**
     * style for the timer
     */
    timerStyle: {
      default: null,
    },
    /**
     * class for the fullscreen button
     */
    fullscreenButtonClass: {
      default: "fullscreen-button-class",
    },
    /**
     * style for the fullscreen button
     */
    fullscreenButtonStyle: {
      default: null,
    },
  },
  data() {
    return {
      showSound: false,
      mouseover: false,
      playing: false,
      showControlsInternal: false,
      started: false,
      loading: false,
      time: 0,
      duration: 0,
      volumeInternal: 1,
      fullscreenInternal: false,
    }
  },
  watch: {
    value(after) {
      if (after && this.$refs.video.paused) {
        this.$refs.video.play()
      } else if (!after && !this.$refs.video.paused) {
        this.$refs.video.pause()
      }
    },

    volume() {
      this.$refs.video.volume = this.volumeComputed
    },
  },
  methods: {
    /**
     * @private
     */
    test(e) {
      console.log("test", e)
    },

    /**
     * @private
     */
    atEnded() {
      if (typeof this.src != "string") {
        /**
         * if src is a array, at the end of the video emits the new src-index
         */
        this.$emit("update:srcIndex", this.srcIndex + 1)
      }
    },

    /**
     * @private
     */
    atFullscreen() {
      if (!this.fullscreenComputed) {
        if (this.$el.requestFullscreen) {
          this.$el.requestFullscreen()
        } else if (this.$el.mozRequestFullScreen) {
          /* Firefox */
          this.$el.mozRequestFullScreen()
        } else if (this.$el.webkitRequestFullscreen) {
          /* Chrome, Safari and Opera */
          this.$el.webkitRequestFullscreen()
        } else if (this.$el.msRequestFullscreen) {
          /* IE/Edge */
          this.$el.msRequestFullscreen()
        }
        this.fullscreenComputed = true

        /**
         * emitted when players go fullscreen
         */
        this.$emit("fullscreen", true)
      } else {
        if (document.exitFullscreen) {
          document.exitFullscreen()
        } else if (document.mozCancelFullScreen) {
          /* Firefox */
          document.mozCancelFullScreen()
        } else if (document.webkitExitFullscreen) {
          /* Chrome, Safari and Opera */
          document.webkitExitFullscreen()
        } else if (document.msExitFullscreen) {
          /* IE/Edge */
          document.msExitFullscreen()
        }

        this.fullscreenComputed = false

        /**
         * emited when players exits fullscreen
         */
        this.$emit("fullscreen", false)
      }
    },

    /**
     * @private
     */
    atVolumechange() {
      this.volume = this.$refs.video.volume
    },

    /**
     * @private
     */
    atVolume() {
      this.showSound = !this.showSound
    },

    /**
     * @private
     */
    atCanplay() {
      this.duration = this.$refs.video.duration

      if (this.started && !this.checkPlaying()) this.$refs.video.play()
    },

    /**
     * @private
     */
    checkPlaying() {
      let result = !!(
        this.$refs.video.currentTime > 0 &&
        !this.$refs.video.paused &&
        !this.$refs.video.ended &&
        this.$refs.video.readyState > 2
      )

      return result
    },

    /**
     * @private
     */
    atTimeupdate() {
      this.time = this.$refs.video.currentTime
    },

    /**
     * @private
     */
    atPlayPause() {
      if (!this.playingComputed) this.$refs.video.play()
      else this.$refs.video.pause()
    },

    /**
     * @private
     */
    atMouseenter() {
      this.mouseover = true
      this.showControlsComputed = true
    },

    /**
     * @private
     */
    atMouseleave() {
      this.mouseover = false
      this.showControlsComputed = false
    },

    /**
     * @private
     */
    play() {
      this.playingComputed = true
      this.started = true
    },

    /**
     * @private
     */
    pause() {
      if (this.time == 0) return
      this.playingComputed = false
    },
  },

  computed: {
    /**
     * @private
     */
    controlsStyleComputed() {
      if (this.controlsStyle != undefined) return this.controlsStyle

      let result = ""

      return result
    },

    /**
     * @private
     */
    volumeComputed: {
      get() {
        if (this.volume != undefined && typeof this.volume == "number")
          return this.volume

        return this.volumeInternal
      },

      set(e) {
        this.$refs.video.volume = e

        if (this.volume != undefined) {
          /**
           * emited when volume is changed
           */
          this.$emit("update:volume", e)
        } else {
          this.volumeInternal = e
        }
      },
    },

    /**
     * @private
     */
    showControlsComputed: {
      get() {
        if (this.controls != undefined) return this.controls

        return this.showControlsInternal
      },

      set(e) {
        if (this.controls != undefined) {
          /**
           * emited when control bar is show/hidden
           */
          this.$emit("update:controls", e)
        } else {
          this.showControlsInternal = e
        }
      },
    },

    /**
     * @private
     */
    playingComputed: {
      get() {
        if (this.value != undefined) return this.value

        return this.playing
      },

      set(e) {
        if (this.value != undefined) {
          this.$emit("input", e)
        } else {
          this.playing = e
        }
      },
    },

    /**
     * @private
     */
    fullscreenComputed: {
      get() {
        return this.fullscreenInternal
      },

      set(e) {
        this.fullscreenInternal = e
      },
    },

    /**
     * @private
     */
    srcComputed() {
      if (typeof this.src == "string") return this.src

      return this.src[this.srcIndex]
    },

    /**
     * @private
     */
    showControlsIf() {
      return (
        (this.showControlsComputed || !this.playingComputed) && this.started
      )
    },

    /**
     * @private
     */
    minutesDuration() {
      let minutes = Math.floor(this.duration / 60)

      let seconds = Math.floor(this.duration - minutes * 60)
      seconds = String(seconds).padStart(2, 0)

      return `${minutes}:${seconds}`
    },

    /**
     * @private
     */
    minutesTime() {
      let minutes = Math.floor(this.time / 60)

      let seconds = Math.floor(this.time - minutes * 60)
      seconds = String(seconds).padStart(2, 0)

      return `${minutes}:${seconds}`
    },

    /**
     * @private
     */
    timeComputed: {
      get() {
        return this.time
      },

      set(e) {
        this.$refs.video.currentTime = e
      },
    },
  },
}
</script>

<style scoped>
.title {
  padding-top: 3px;
  padding-left: 5px;
  padding-right: 5px;
  padding-bottom: 20px;
  width: calc(100% - 6px);
  align-self: start;
  font-size: 18px;
  letter-spacing: 1px;
  position: absolute;
  top: 0;
  color: white;
  background-image: linear-gradient(rgba(0, 0, 0, 0.3), rgba(0, 0, 0, 0));
}

.start-button {
  width: auto;
  height: auto;
  border: 8px solid white;
  border-radius: 1rem;
  background-color: rgba(0, 0, 0, 0.3);
  cursor: pointer;
}

.start-button:hover {
  background-color: rgba(0, 0, 0, 0.5);
}

.size-124 {
  font-size: 124px;
}

.material-icons {
  vertical-align: bottom;
}

.transparent {
  opacity: 0;
}

.placeholder {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
}

.vue-video-player video {
  width: 100%;
  height: 100%;
}

.vue-video-player {
  position: relative;
  font-size: 0;
  color: white;
  display: block;
}

.timer {
  width: auto;
}

.vue-video-center {
  color: white;
  bottom: 0;
  left: 0;
  right: 0;
  top: 0;
  justify-content: center;
  align-items: center;
  display: flex;
  font-size: 12px;
  position: absolute;
}

.fullscreen-button-class {
  margin-left: auto;
}

.spin {
  position: absolute;
  top: calc(50% - 62px);
  left: calc(50% - 62px);
  -webkit-animation: rotation 2s infinite linear;
}

@-webkit-keyframes rotation {
  from {
    -webkit-transform: rotate(0deg);
  }
  to {
    -webkit-transform: rotate(359deg);
  }
}

input[type="range"].vue-player-time-control {
  position: absolute;
  top: -5px;
  left: 0;
  padding-top: 0;
}

/* Special styling for WebKit/Blink */
input[type="range"].vue-player-time-control::-webkit-slider-thumb {
  height: 10px;
  width: 10px;
  margin-top: -3px;
  border-radius: 50%;
}

/* All the same stuff for Firefox */
input[type="range"].vue-player-time-control::-moz-range-thumb {
  height: 10px;
  width: 10px;
  margin-top: -3px;
  border-radius: 50%;
}

/* All the same stuff for IE */
input[type="range"].vue-player-time-control::-ms-thumb {
  height: 10px;
  width: 10px;
  margin-top: -3px;
  border-radius: 50%;
}
</style>
