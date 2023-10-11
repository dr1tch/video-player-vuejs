<template>
  <button :class="defaultClass"><slot /></button>
</template>

<script>
export default {
  name: "player-button",
  props: {
    show: Boolean,
  },
  computed: {
    defaultClass() {
      if (this.$attrs.class) return this.$attrs.class
      return "player-button"
    },
  },
}
</script>

<style scoped>
.player-button {
  width: 35px;
  border: 0px solid white;
  color: white;
  background-color: transparent;
  outline: 0px;
}
</style>
