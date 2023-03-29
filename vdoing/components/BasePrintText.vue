<template>
  <div class="container">
    <span>{{ printText }}|</span>
  </div>
</template>

<script>
export default {
  name: "Vue2PracticeApp",
  props: { text: String },
  data() {
    return {
      count: 1,
      timer: null,
      index: 0,
      printText: "",
    };
  },

  methods: {
    setForward(text) {
      let timer = null;
      timer = setInterval(() => {
        if (this.count == text.length + 1) {
          clearInterval(timer);
          timer = null;
          this.setBack(text);
        }

        this.printText = text.substr(0, this.count);
        this.count++;
      }, 200);
    },

    setBack(text) {
      this.timer = setInterval(() => {
        if (this.count == 0) {
          clearInterval(this.timer);
          this.timer = null;
        }

        this.printText = text.substr(0, this.count);
        this.count--;
      }, 100);
    },
  },

  watch: {
    timer: {
      handler(newVal) {
        if (!newVal) {
          // 重新请求
          this.$emit("refetch", true);
        }
      },
    },
    text: {
      handler() {
        let printText = this.text;
        this.setForward(printText);
      },
    },
  },
};
</script>

<style scoped>
.container {
  line-height: 2rem;
  font-size: 1.4rem;
  max-width: 70%;
  margin: 0 auto;
  white-space: break-spaces;
}
</style>
