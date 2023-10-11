<template>
  <section class="section">
    <vue-player
      src="http://techslides.com/demos/sample-videos/small.mp4"
      video-placeholder-src="http://techslides.com/demos/sample-videos/small.mp4"
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
