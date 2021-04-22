<template>
  <div class="text-search-container">
    <p class="box-title">
      Color-based search
    </p>

    <input v-model="selectedColor" type="color" id="color" name="color" />

    <div class="canvas-wrapper">
      <canvas
        @mousedown="startPainting"
        @mouseup="finishedPainting"
        @mousemove="draw"
        width="512"
        height="512"
        ref="canvas"
      ></canvas>
    </div>

    <button @click.prevent="search">
      Search
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedColor: "#00000",
      painting: false,
    };
  },
  methods: {
    search() {
      console.log(
        "searching for class-labelled image",
        this.$refs.canvas
          .getContext("2d")
          .getImageData(0, 0, this.$refs.canvas.width, this.$refs.canvas.height)
      );
    },

    getMousePos(evt) {
      var rect = this.$refs.canvas.getBoundingClientRect(), // abs. size of element
        scaleX = this.$refs.canvas.width / rect.width, // relationship bitmap vs. element for X
        scaleY = this.$refs.canvas.height / rect.height; // relationship bitmap vs. element for Y

      return {
        x: (evt.clientX - rect.left) * scaleX, // scale mouse coordinates after they have
        y: (evt.clientY - rect.top) * scaleY, // been adjusted to be relative to element
      };
    },
    startPainting(e) {
      this.painting = true;
      this.draw(e);
    },
    finishedPainting() {
      this.painting = false;
      this.$refs.canvas.getContext("2d").beginPath();
    },
    draw(e) {
      if (!this.painting) return;

      this.$refs.canvas.getContext("2d").lineWidth = 30;
      // TODO: convert color to CIE Lab
      this.$refs.canvas.getContext("2d").strokeStyle = this.selectedColor;
      this.$refs.canvas.getContext("2d").lineCap = "round";

      const mousePos = this.getMousePos(e);

      this.$refs.canvas.getContext("2d").lineTo(mousePos.x, mousePos.y);
      this.$refs.canvas.getContext("2d").stroke();

      this.$refs.canvas.getContext("2d").beginPath();
      this.$refs.canvas.getContext("2d").moveTo(mousePos.x, mousePos.y);
    },
  },
};
</script>

<style lang="scss" scoped>
.text-search-container {
  display: flex;
  flex-direction: column;
  min-height: 0;

  .box-title {
    margin-bottom: 1rem;
  }

  button {
    margin-top: 1rem;
  }

  .canvas-wrapper {
    min-height: 0;
    margin-top: 1rem;

    canvas {
      border: lightgray thin solid;
      max-height: 100%;
      max-width: 100%;
      min-height: 0;
    }
  }
}
</style>
