<template>
  <div class="text-search-container">
    <p class="box-title">
      Color-based search
    </p>

    <input v-model="selectedColor" type="color" id="color" name="color" />

    <div
      class="canvas-wrapper"
      @mousedown="mouseDown = true"
      @mouseup="mouseDown = false"
    >
      <div class="row" v-for="y in colors.length" :key="y">
        <div
          v-for="x in colors[y - 1].length"
          :key="x"
          @mousedown="setColor(y - 1, x - 1)"
          @mouseenter="setColorIfMouseDown(y - 1, x - 1)"
          :style="{
            backgroundColor: `rgb(${colors[y - 1][x - 1][0]},${
              colors[y - 1][x - 1][1]
            },${colors[y - 1][x - 1][2]})`,
          }"
        ></div>
      </div>
    </div>

    <div class="button-container">
      <button @click.prevent="$emit('clear')">
        Clear
      </button>

      <button @click.prevent="$emit('search')">
        Search
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedColor: "#00000",
      mouseDown: false,
    };
  },

  props: ["colors"],

  methods: {
    convertHex(hexCode) {
      var hex = hexCode.replace("#", "");

      if (hex.length === 3) {
        hex = hex[0] + hex[0] + hex[1] + hex[1] + hex[2] + hex[2];
      }

      var r = parseInt(hex.substring(0, 2), 16),
        g = parseInt(hex.substring(2, 4), 16),
        b = parseInt(hex.substring(4, 6), 16);

      return [r, g, b];
    },

    setColor(y, x) {
      const rgbColor = this.convertHex(this.selectedColor);
      // console.log("setting color for", y, x, rgbColor);
      this.$emit("setColor", y, x, rgbColor);
    },

    setColorIfMouseDown(y, x) {
      if (this.mouseDown) {
        const rgbColor = this.convertHex(this.selectedColor);
        // console.log("setting color for", y, x, rgbColor);
        this.$emit("setColor", y, x, rgbColor);
      }
    },

    // getMousePos(evt) {
    //   var rect = this.$refs.canvas.getBoundingClientRect(), // abs. size of element
    //     scaleX = this.$refs.canvas.width / rect.width, // relationship bitmap vs. element for X
    //     scaleY = this.$refs.canvas.height / rect.height; // relationship bitmap vs. element for Y

    //   return {
    //     x: (evt.clientX - rect.left) * scaleX, // scale mouse coordinates after they have
    //     y: (evt.clientY - rect.top) * scaleY, // been adjusted to be relative to element
    //   };
    // },
    // startPainting(e) {
    //   this.painting = true;
    //   this.draw(e);
    // },
    // finishedPainting() {
    //   this.painting = false;
    //   this.$refs.canvas.getContext("2d").beginPath();
    // },
    // draw(e) {
    //   if (!this.painting) return;

    //   this.$refs.canvas.getContext("2d").lineWidth = 30;
    //   // TODO: convert color to CIE Lab
    //   this.$refs.canvas.getContext("2d").strokeStyle = this.selectedColor;
    //   this.$refs.canvas.getContext("2d").lineCap = "round";

    //   const mousePos = this.getMousePos(e);

    //   this.$refs.canvas.getContext("2d").lineTo(mousePos.x, mousePos.y);
    //   this.$refs.canvas.getContext("2d").stroke();

    //   this.$refs.canvas.getContext("2d").beginPath();
    //   this.$refs.canvas.getContext("2d").moveTo(mousePos.x, mousePos.y);
    // },
  },
};
</script>

<style lang="scss" scoped>
.text-search-container {
  display: flex;
  flex-direction: column;
  // min-height: 0;

  .box-title {
    font-weight: 600;
  }

  .button-container {
    display: flex;
    margin-top: 0.25rem;

    button {
      flex-grow: 1;
    }
  }

  .canvas-wrapper {
    margin-top: 0.25rem;

    // canvas {
    //   border: lightgray thin solid;
    //   max-height: 100%;
    //   max-width: 100%;
    //   min-height: 0;
    // }

    .row {
      width: 100%;
      display: flex;

      div {
        width: calc(100% / 8);
        padding-top: calc(100% / 8);
        // height: calc(100% / 8);
        border: thin gainsboro solid;
      }
    }
  }
}
</style>
