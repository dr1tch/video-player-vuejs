<template>
  <section class="section">
    <VideoPlayer
      src="https://d2of6bhnpl91ni.cloudfront.net/cms/Tutorial-video-maker-3a570d8068.mp4"
    >
    </VideoPlayer>
  </section>
</template>

<script>
import VideoPlayer from "./components/player.vue"

export default {
  components: {
    VideoPlayer,
  },
}
</script>
