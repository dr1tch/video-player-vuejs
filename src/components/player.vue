<template>
  <div
    class="video-container"
    allowFullscreen
    allowPictureOnPicture
    ref="videoContainer"
    :class="videoContainerClassList"
    tabindex="0"
    @keydown="handleKeyDown"
    data-volume-level="high"
  >
    <div class="video-controls-container" :class="{ isPaused: 'pause' }">
      <img class="thumbnail-img" ref="thumbnailImg" />
      <div
        class="timeline-container"
        ref="timelineContainer"
        @mousemove="handleTimelineUpdate"
        @mousedown="toggleScrubbing"
        @mouseover="handleScrubbingOnMouseup"
      >
        <div class="timeline">
          <!-- <img class="preview-img" ref="previewImg" /> -->
          <div class="thumb-indicator"></div>
        </div>
      </div>
      <div class="controls">
        <button
          class="play-pause-button"
          ref="playPauseBtn"
          @click="togglePlay"
          :title="isPaused ? 'Play' : 'Pause'"
        >
          <svg
            v-if="isPaused || isFinished"
            class="play-icon"
            viewBox="0 0 24 24"
          >
            <path fill="currentColor" d="M8,5.14V19.14L19,12.14L8,5.14Z" />
          </svg>
          <svg v-else class="pause-icon" viewBox="0 0 24 24">
            <path fill="currentColor" d="M14,19H18V5H14M6,19H10V5H6V19Z" />
          </svg>
        </button>
        <div class="volume-container">
          <button
            class="mute-btn"
            ref="muteBtn"
            @click="toggleMute"
            :title="isMuted ? 'Unmute' : 'Mute'"
          >
            <svg v-if="isMuted" class="volume-muted-icon" viewBox="0 0 24 24">
              <path
                fill="currentColor"
                d="M12,4L9.91,6.09L12,8.18M4.27,3L3,4.27L7.73,9H3V15H7L12,20V13.27L16.25,17.53C15.58,18.04 14.83,18.46 14,18.7V20.77C15.38,20.45 16.63,19.82 17.68,18.96L19.73,21L21,19.73L12,10.73M19,12C19,12.94 18.8,13.82 18.46,14.64L19.97,16.15C20.62,14.91 21,13.5 21,12C21,7.72 18,4.14 14,3.23V5.29C16.89,6.15 19,8.83 19,12M16.5,12C16.5,10.23 15.5,8.71 14,7.97V10.18L16.45,12.63C16.5,12.43 16.5,12.21 16.5,12Z"
              />
            </svg>
            <div v-else>
              <svg class="volume-high-icon" viewBox="0 0 24 24">
                <path
                  fill="currentColor"
                  d="M14,3.23V5.29C16.89,6.15 19,8.83 19,12C19,15.17 16.89,17.84 14,18.7V20.77C18,19.86 21,16.28 21,12C21,7.72 18,4.14 14,3.23M16.5,12C16.5,10.23 15.5,8.71 14,7.97V16C15.5,15.29 16.5,13.76 16.5,12M3,9V15H7L12,20V4L7,9H3Z"
                />
              </svg>
              <svg class="volume-low-icon" viewBox="0 0 24 24">
                <path
                  fill="currentColor"
                  d="M5,9V15H9L14,20V4L9,9M18.5,12C18.5,10.23 17.5,8.71 16,7.97V16C17.5,15.29 18.5,13.76 18.5,12Z"
                />
              </svg>
            </div>
          </button>
          <input
            class="volume-slider"
            type="range"
            min="0"
            max="1"
            step="any"
            ref="volumeSlider"
            :value="volume"
            @change="handleVolumeChange"
          />
        </div>
        <div class="duration-container">
          <div class="current-time">{{ currentTime }}</div>
          /
          <div class="total-time">{{ duration }}</div>
        </div>
        <button
          @click="changePlaybackSpeed"
          ref="speedBtn"
          class="speed-btn wide"
        >
          {{ playbackRate }}
        </button>
        <button
          title="Picture in picture"
          class="mini-player-btn"
          ref="miniPlayerBtn"
          @click="toggleMiniPlayerMode"
        >
          <svg viewBox="0 0 24 24">
            <path
              fill="currentColor"
              d="M21 3H3c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h18c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 16H3V5h18v14zm-10-7h9v6h-9z"
            />
          </svg>
        </button>
        <button
          class="theater-btn"
          ref="theaterBtn"
          @click="toggleTheaterMode"
          :title="isTheaterMode ? 'Default Mode' : 'Theater Mode'"
        >
          <svg v-if="isTheaterMode" class="tall" viewBox="0 0 24 24">
            <path
              fill="currentColor"
              d="M19 6H5c-1.1 0-2 .9-2 2v8c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2zm0 10H5V8h14v8z"
            />
          </svg>
          <svg v-else class="wide" viewBox="0 0 24 24">
            <path
              fill="currentColor"
              d="M19 7H5c-1.1 0-2 .9-2 2v6c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V9c0-1.1-.9-2-2-2zm0 8H5V9h14v6z"
            />
          </svg>
        </button>
        <button
          v-if="!isFullscreenMode"
          class="full-screen-btn"
          ref="fullscreenBtn"
          @click="openFullscreenMode"
          title="Full screen (f)"
        >
          <svg class="open" viewBox="0 0 24 24">
            <path
              fill="currentColor"
              d="M7 14H5v5h5v-2H7v-3zm-2-4h2V7h3V5H5v5zm12 7h-3v2h5v-5h-2v3zM14 5v2h3v3h2V5h-5z"
            />
          </svg>
        </button>
        <button
          v-else
          class="full-screen-btn"
          ref="fullscreenBtn"
          @click="closeFullscreenMode"
          title="Exit full screen (f)"
        >
          <svg class="close" viewBox="0 0 24 24">
            <path
              fill="currentColor"
              d="M5 16h3v3h2v-5H5v2zm3-8H5v2h5V5H8v3zm6 11h2v-3h3v-2h-5v5zm2-11V5h-2v5h5V8h-3z"
            />
          </svg>
        </button>
      </div>
    </div>
    <video
      @click="togglePlay"
      :src="src"
      ref="video"
      @pause="removePaused"
      @play="removePaused"
      @ended="handleEnded"
      @loadedmetadata="getDuration"
      @timeupdate="handleTimeUpdate"
    />
  </div>
  <ul>
    <li>isPaused: {{ isPaused }}</li>
    <li>isFinished: {{ isFinished }}</li>
    <li>isTheaterMode: {{ isTheaterMode }}</li>
    <li>isFullscreenMode: {{ isFullscreenMode }}</li>
    <li>isMiniPlayerMode: {{ isMiniPlayerMode }}</li>
    <li>isMuted: {{ isMuted }}</li>
    <li>isScrubbing: {{ isScrubbing }}</li>
    <li>duration: {{ duration }}</li>
    <li>currentTime: {{ currentTime }}</li>
    <li>videoDuration: {{ videoDuration }}</li>
    <li>realtime: {{ realtime }}</li>
  </ul>
</template>

<script>
const leadingZeroFormatter = new Intl.NumberFormat(undefined, {
  minimumIntegerDigits: 2,
})
export default {
  props: {
    src: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      isPaused: true,
      isFinished: false,
      isFullscreenMode: window.innerHeight == screen.height,
      isTheaterMode: false,
      isMiniPlayerMode: false,
      isMuted: false,
      volume: 0.5,
      duration: "0:00",
      videoDuration: 0,
      currentTime: "0:00",
      realtime: 0,
      playbackRate: "1x",
      isScrubbing: false,
      wasPaused: null,
    }
  },
  methods: {
    togglePlay(e) {
      this.$refs.video && this.$refs.video?.paused
        ? this.$refs.video.play()
        : this.$refs.video.pause()
      this.isPaused = this.$refs.video?.paused
    },
    toggleTheaterMode() {
      this.isTheaterMode = !this.isTheaterMode
    },
    closeFullscreenMode() {
      const element = this.$refs.videoContainer
      element.focus()
      if (document.fullscreenElement) {
        document.exitFullscreen().catch((err) => {
          console.error(err)
        })
      }
      this.isFullscreenMode = false
    },
    toggleMiniPlayerMode() {
      if (this.isMiniPlayerMode) {
        document.exitPictureInPicture()
        this.isMiniPlayerMode = false
      } else {
        this.$refs.video.requestPictureInPicture()
        this.isMiniPlayerMode = true
      }
    },
    openFullscreenMode() {
      const element = this.$refs.videoContainer

      if (element) {
        if (element.requestFullscreen) {
          element
            .requestFullscreen()
            .then(() => {
              this.isFullscreenMode = true
              // element.focus()
            })
            .catch((err) => {
              console.error(err)
            })
        } else if (element.webkitRequestFullscreen) {
          element
            .webkitRequestFullscreen()
            .then(() => {
              this.isFullscreenMode = true
              // element.focus()
            })
            .catch((err) => {
              console.error(err)
            })
        } else if (element.mozRequestFullScreen) {
          element
            .mozRequestFullScreen()
            .then(() => {
              this.isFullscreenMode = true
              // element.focus()
            })
            .catch((err) => {
              console.error(err)
            })
        } else if (element.msRequestFullscreen) {
          element
            .msRequestFullscreen()
            .then(() => {
              this.isFullscreenMode = true
              // element.focus()
            })
            .catch((err) => {
              console.error(err)
            })
        }
      }
    },
    handleVolumeChange(e) {
      this.$refs.video.volume = e.target.value
      this.$refs.video.muted = Number(e.target.value) === 0
    },
    handleEnded() {
      this.isFinished = true
    },
    toggleMute() {
      this.$refs.video.focus()
      if (this.$refs.video) {
        this.$refs.video.muted = !this.isMuted
        this.isMuted = !this.isMuted
      }
    },
    formatDuration(time) {
      const seconds = Math.floor(time % 60)
      const minutes = Math.floor(time / 60) % 60
      const hours = Math.floor(time / 3600)
      if (hours === 0) {
        return `${minutes}:${leadingZeroFormatter.format(seconds)}`
      } else {
        return `${hours}:${leadingZeroFormatter.format(
          minutes
        )}:${leadingZeroFormatter.format(seconds)}`
      }
    },
    getDuration() {
      this.duration = this.formatDuration(this.$refs.video.duration)
      this.videoDuration = this.$refs.video.duration
    },
    handleTimeUpdate() {
      this.currentTime = this.formatDuration(this.$refs.video.currentTime)
      this.realtime = this.$refs.video.currentTime

      const percent = Number(this.realtime) / Number(this.videoDuration)
      this.$refs.timelineContainer.style.setProperty(
        "--progress-position",
        percent
      )
    },
    skip(duration = 5) {
      this.$refs.video.currentTime += duration
    },
    // Timeline
    handleTimelineUpdate(e) {
      const rect = this.$refs.timelineContainer.getBoundingClientRect()
      const percent =
        Math.min(Math.max(0, e.x - rect.x), rect.width) / rect.width
      const previewImgNumber = Math.max(
        1,
        Math.floor((percent * this.duration) / 10)
      )
      this.$refs.timelineContainer.style.setProperty(
        "--preview-position",
        percent
      )

      if (this.isScrubbing) {
        e.preventDefault()
        this.$refs.timelineContainer.style.setProperty(
          "--progress-position",
          percent
        )
      }
    },
    toggleScrubbing(e) {
      const rect = this.$refs.timelineContainer.getBoundingClientRect()
      const percent =
        Math.min(Math.max(0, e.x - rect.x), rect.width) / rect.width
      this.isScrubbing = (e.buttons & 1) === 1
      if (this.isScrubbing) {
        this.wasPaused = this.$refs.video.paused
        this.$refs.video.pause()
      } else {
        this.realtime = percent * this.videoDuration
        this.$refs.video.currentTime = percent * this.videoDuration

        if (!this.wasPaused) this.$refs.video.play()
      }

      this.handleTimelineUpdate(e)
    },
    handleScrubbingOnMouseup(e) {
      if (this.isScrubbing) {
        this.toggleScrubbing(e)
      }
    },
    handleScrubbingOnMousemove(e) {
      if (this.isScrubbing) {
        this.handleTimelineUpdate(e)
      }
    },
    changePlaybackSpeed() {
      let newPlaybackRate = this.$refs.video.playbackRate + 0.25
      if (newPlaybackRate > 2) newPlaybackRate = 0.25
      if (this.$refs.video) this.$refs.video.playbackRate = newPlaybackRate
      this.playbackRate = `${newPlaybackRate}x`
    },
    handleMuted() {
      let volumeLevel = null
      this.$refs.volumeSlider.value = this.$refs.video.volume
      if (Number(this.$refs.video.volume) <= 0 || this.$refs.video.muted) {
        volumeLevel = "muted"
        this.$refs.volumeSlider.value = 0
        this.volume = 0
        this.isMuted = true
      } else if (this.$refs.video.volume >= 0.5) {
        volumeLevel = "high"
      } else {
        volumeLevel = "low"
      }
      this.$refs.videoContainer.dataset.volumeLevel = volumeLevel
    },
    handleKeyDown(e) {
      const tagName = document.activeElement.tagName.toLowerCase()
      const element = this.$refs.videoContainer

      if (tagName === "input") return

      switch (e.key.toLowerCase()) {
        case " ":
          if (tagName === "button") return
        case "k":
          this.togglePlay()
          break
        case "escape":
          // element.focus()
          console.log("escape clicked")
          this.closeFullscreenMode()
          break
        case "f":
          // element.focus()
          this.isFullscreenMode
            ? this.closeFullscreenMode()
            : this.openFullscreenMode()
          break
        case "t":
          this.toggleTheaterMode()
          break
        case "i":
          this.toggleMiniPlayerMode()
          break
        case "m":
          this.toggleMute()
          break
        case "arrowleft":
        case "j":
          this.skip(-5)
          break
        case "arrowright":
        case "l":
          this.skip(5)
          break
      }
    },
  },
  mounted() {
    // Add an event listener to track focus changes
    window.addEventListener("keydown", this.handleKeyDown)
    this.$refs.video.addEventListener("volumechange", this.handleMuted)
    document.addEventListener("mousemove", this.handleScrubbingOnMousemove)
  },
  destroyed() {
    // Don't forget to remove the event listener when the component is destroyed
    window.removeEventListener("keydown", this.handleKeyDown)
    this.$refs.video.removeEventListener("volumechange", this.handleMuted)
    document.removeEventListener("mousemove", this.handleScrubbingOnMousemove)
  },

  computed: {
    videoContainerClassList() {
      return {
        pause: this.isPaused,
        theater: this.isTheaterMode,
        scrubbing: this.isScrubbing,
      }
    },
  },
}
</script>

<style scoped>
.video-container {
  width: 90%;
  max-width: 1000px;
  display: flex;
  justify-content: center;
  margin-inline: auto;
  position: relative;
  background-color: black;
}

.video-container.theater,
.video-container.full-screen {
  max-width: initial;
  width: 100%;
}

.video-container.theater {
  max-height: 90vh;
}

.video-container.full-screen {
  max-height: 100vh;
}

video {
  width: 100%;
}

.video-controls-container {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  color: white;
  z-index: 100;
  /* opacity: 0; */
  transition: opacity 150ms ease-in-out;
}

.video-controls-container::before {
  content: "";
  position: absolute;
  bottom: 0;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.75), transparent);
  width: 100%;
  aspect-ratio: 6/1;
  z-index: -1;
  pointer-events: none;
}

.video-container:hover .video-controls-container,
.video-container:focus-within .video-controls-container,
.video-container.pause .video-controls-container {
  opacity: 1;
}

.video-controls-container .controls {
  display: flex;
  gap: 0.5rem;
  padding: 0.25rem;
  align-items: center;
}

.video-controls-container .controls button {
  background: none;
  border: none;
  color: inherit;
  padding: 0;
  height: 30px;
  width: 30px;
  font-size: 1.1rem;
  cursor: pointer;
  opacity: 0.85;
  transition: opacity 150ms ease-in-out;
}

.video-controls-container .controls button:hover {
  opacity: 1;
}

.volume-high-icon,
.volume-low-icon,
.volume-muted-icon {
  display: none;
}

.video-container[data-volume-level="high"] .volume-high-icon {
  display: block;
}

.video-container[data-volume-level="low"] .volume-low-icon {
  display: block;
}

.video-container[data-volume-level="muted"] .volume-muted-icon {
  display: block;
}

.volume-container {
  display: flex;
  align-items: center;
}

.volume-slider {
  width: 0;
  transform-origin: left;
  transform: scaleX(0);
  transition: width 150ms ease-in-out, transform 150ms ease-in-out;
}

.volume-container:hover .volume-slider,
.volume-slider:focus-within {
  width: 100px;
  transform: scaleX(1);
}

.duration-container {
  display: flex;
  align-items: center;
  gap: 0.25rem;
  flex-grow: 1;
}

.video-container.captions .captions-btn {
  border-bottom: 3px solid red;
}

.video-controls-container .controls button.wide-btn {
  width: 50px;
}

.timeline-container {
  height: 7px;
  margin-inline: 0.5rem;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.timeline {
  background-color: rgba(100, 100, 100, 0.5);
  height: 3px;
  width: 100%;
  position: relative;
}

.timeline::before {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  right: calc(100% - var(--preview-position) * 100%);
  background-color: rgb(150, 150, 150);
  display: none;
}

.timeline::after {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  right: calc(100% - var(--progress-position) * 100%);
  background-color: red;
}

.timeline .thumb-indicator {
  --scale: 0;
  position: absolute;
  transform: translateX(-50%) scale(var(--scale));
  height: 200%;
  top: -50%;
  left: calc(var(--progress-position) * 100%);
  background-color: red;
  border-radius: 50%;
  transition: transform 150ms ease-in-out;
  aspect-ratio: 1 / 1;
}

.timeline .preview-img {
  position: absolute;
  height: 80px;
  aspect-ratio: 16 / 9;
  top: -1rem;
  transform: translate(-50%, -100%);
  left: calc(var(--preview-position) * 100%);
  border-radius: 0.25rem;
  border: 2px solid white;
  display: none;
}

.thumbnail-img {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
  display: none;
}

.video-container.scrubbing .thumbnail-img {
  display: block;
}

.video-container.scrubbing .preview-img,
.timeline-container:hover .preview-img {
  display: block;
}

.video-container.scrubbing .timeline::before,
.timeline-container:hover .timeline::before {
  display: block;
}

.video-container.scrubbing .thumb-indicator,
.timeline-container:hover .thumb-indicator {
  --scale: 1;
}

.video-container.scrubbing .timeline,
.timeline-container:hover .timeline {
  height: 100%;
}
</style>
