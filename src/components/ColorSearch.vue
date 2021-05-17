<template>
  <div class="text-search-container">
    <p class="box-title">
      Color-based search
    </p>

    <input v-model="selectedColor" type="color" id="color" name="color" />

    <div class="canvas-wrapper">
      <!-- <canvas
        @mousedown="startPainting"
        @mouseup="finishedPainting"
        @mousemove="draw"
        width="512"
        height="512"
        ref="canvas"
      ></canvas> -->
      <!-- <div class="row" v-for="y in colorsRGB.length" :key="y">
        <div
          v-for="x in y.length"
          :key="x"
          :style="{
            backgroundColor: `rgb(${colorsRGB[y][x][0]},${colorsRGB[y][x][1]},${colorsRGB[y][x][2]}`,
          }"
          @click="setColor(y, x)"
        >
          {{ y }}-{{ x }}
        </div>
      </div> -->
      <div class="row" v-for="y in colorsRGB.length" :key="y">
        <div
          v-for="x in colorsRGB[y - 1].length"
          :key="x"
          @click="setColor(y - 1, x - 1)"
          :style="{
            backgroundColor: `rgb(${colorsRGB[y - 1][x - 1][0]},${
              colorsRGB[y - 1][x - 1][1]
            },${colorsRGB[y - 1][x - 1][2]})`,
          }"
        ></div>
      </div>
    </div>

    <button @click.prevent="search">
      Search
    </button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      selectedColor: "#00000",
      painting: false,
      colorsRGB: [
        [
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
        ],
        [
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
        ],
        [
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
        ],
        [
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
        ],
        [
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
        ],
        [
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
        ],
        [
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
        ],
        [
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
          [255, 255, 255],
        ],
      ],
    };
  },

  computed: {
    colorsLAB() {
      return this.colorsRGB.map((r) => r.map((c) => this.rgb2lab(c)));
    },
  },
  methods: {
    async search() {
      console.log(
        "searching for class-labelled image",
        //   this.$refs.canvas
        //     .getContext("2d")
        //     .getImageData(0, 0, this.$refs.canvas.width, this.$refs.canvas.height)

        this.colorsLAB
      );

      this.$emit("fetchStart");

      const res = await (
        await axios.post(
          `http://localhost:9191/SearchColorMilvus?topk=500`,
          this.colorsLAB
        )
      ).data;

      console.log("result", res);

      // console.log(res);
      // this.boxes.forEach((b) => {
      //   // get the 4 coords
      // });

      const pictureIds = res.map((e) => e.pictureId);
      // console.log(pictureIds);

      this.$emit("fetchEnd", pictureIds);
    },

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

    rgb2lab(rgb) {
      var r = rgb[0] / 255,
        g = rgb[1] / 255,
        b = rgb[2] / 255,
        x,
        y,
        z;

      r = r > 0.04045 ? Math.pow((r + 0.055) / 1.055, 2.4) : r / 12.92;
      g = g > 0.04045 ? Math.pow((g + 0.055) / 1.055, 2.4) : g / 12.92;
      b = b > 0.04045 ? Math.pow((b + 0.055) / 1.055, 2.4) : b / 12.92;

      x = (r * 0.4124 + g * 0.3576 + b * 0.1805) / 0.95047;
      y = (r * 0.2126 + g * 0.7152 + b * 0.0722) / 1.0;
      z = (r * 0.0193 + g * 0.1192 + b * 0.9505) / 1.08883;

      x = x > 0.008856 ? Math.pow(x, 1 / 3) : 7.787 * x + 16 / 116;
      y = y > 0.008856 ? Math.pow(y, 1 / 3) : 7.787 * y + 16 / 116;
      z = z > 0.008856 ? Math.pow(z, 1 / 3) : 7.787 * z + 16 / 116;

      return [116 * y - 16, 500 * (x - y), 200 * (y - z)];
    },

    setColor(y, x) {
      const rgbColor = this.convertHex(this.selectedColor);
      console.log("setting color for", y, x, rgbColor);
      this.$set(this.colorsRGB[y], x, rgbColor);
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
        width: 2rem;
        height: 2rem;
        border: thin gainsboro solid;
      }
    }
  }
}
</style>
