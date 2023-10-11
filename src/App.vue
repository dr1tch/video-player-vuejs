<template>
  <section class="section">
    <vue-player
      src="https://d2of6bhnpl91ni.cloudfront.net/cms/Tutorial-video-maker-3a570d8068.mp4"
    >
    </vue-player>
  </section>
</template>

<script>
import vuePlayer from "./components/player.vue"

export default {
  components: {
    vuePlayer,
  },
}
</script>
