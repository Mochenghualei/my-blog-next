<template>
  <div class="scroll-progress" :style="{ width: progressWidth + '%' }"></div>
</template>

<script>
export default {
  name: "BaseTopProgressBar",
  data() {
    return {
      scrollTop: 0,
    }
  },
  mounted() {
    window.addEventListener('scroll', this.handleScroll)
  },
  beforeDestroy() {
    window.removeEventListener('scroll', this.handleScroll)
  },
  methods: {
    handleScroll() {
      const scrollTop = document.documentElement.scrollTop || document.body.scrollTop
      const scrollHeight = document.documentElement.scrollHeight || document.body.scrollHeight
      const clientHeight = document.documentElement.clientHeight || document.body.clientHeight
      const percent = (scrollTop / (scrollHeight - clientHeight)) * 100
      this.scrollTop = percent
    },
  },
  computed: {
    progressWidth() {
      return Math.min(this.scrollTop, 100)
    },
  },
}
</script>

<style scoped>
.scroll-progress {
  height: 4px;
  position: fixed;
  top: 0;
  left: 0;
  background-color: #ddd;
  background-image: linear-gradient(to right, #4da6c9, #4da6c9);
  z-index: 999;
  transition: width 0.3s;
}
</style>