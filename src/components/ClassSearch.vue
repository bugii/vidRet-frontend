<template>
  <div class="text-search-container">
    <p class="box-title">
      Class-based search
    </p>

    <select v-model="selectedClass" name="classids" id="classids">
      <option v-for="(className, index) in classes" :key="index" :value="index">
        {{ className }}
      </option>
    </select>

    <div class="canvas-wrapper">
      <canvas
        ref="canvas"
        width="512"
        height="512"
        @mousedown="startPainting"
        @mouseup="finishedPainting"
        @mousemove="draw"
      >
      </canvas>

      <canvas
        ref="overlayCanvas"
        width="512"
        height="512"
        class="overlay-canvas"
      >
      </canvas>
    </div>

    <button @click.prevent="search">
      Search
    </button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  components: {},
  data() {
    return {
      painting: false,
      drawingBox: {
        x: 0,
        y: 0,
        width: 0,
        height: 0,
        classId: null,
      },
      boxes: [],
      selectedClass: null,
      classes: [
        "BG",
        "person",
        "bicycle",
        "car",
        "motorcycle",
        "airplane",
        "bus",
        "train",
        "truck",
        "boat",
        "traffic light",
        "fire hydrant",
        "stop sign",
        "parking meter",
        "bench",
        "bird",
        "cat",
        "dog",
        "horse",
        "sheep",
        "cow",
        "elephant",
        "bear",
        "zebra",
        "giraffe",
        "backpack",
        "umbrella",
        "handbag",
        "tie",
        "suitcase",
        "frisbee",
        "skis",
        "snowboard",
        "sports ball",
        "kite",
        "baseball bat",
        "baseball glove",
        "skateboard",
        "surfboard",
        "tennis racket",
        "bottle",
        "wine glass",
        "cup",
        "fork",
        "knife",
        "spoon",
        "bowl",
        "banana",
        "apple",
        "sandwich",
        "orange",
        "broccoli",
        "carrot",
        "hot dog",
        "pizza",
        "donut",
        "cake",
        "chair",
        "couch",
        "potted plant",
        "bed",
        "dining table",
        "toilet",
        "tv",
        "laptop",
        "mouse",
        "remote",
        "keyboard",
        "cell phone",
        "microwave",
        "oven",
        "toaster",
        "sink",
        "refrigerator",
        "book",
        "clock",
        "vase",
        "scissors",
        "teddy bear",
        "hair drier",
        "toothbrush",
      ],
    };
  },

  computed: {
    colors() {
      const nrOfClasses = this.classes.length;
      const colors = this.classes.map((className, i) => {
        return this.selectColor(i, nrOfClasses);
      });
      return colors;
    },
  },

  methods: {
    async search() {
      console.log("searching for the objects", this.boxes);
      this.$emit("fetchStart");

      // convert to 2D array
      const arr = [...Array(8)].map(() => Array(8).fill(Array(81).fill(0)));
      const image = [...Array(512)].map(() => Array(512).fill(0));

      // console.log(arr);
      // console.log(image);

      // console.log(this.boxes);
      // loop over each individual pixel and check if we find any drawn objects in that pixel

      // arr.forEach((yArr, y) => {
      //   yArr.forEach((xArr, x) => {
      //     console.log(y, x);
      //   });
      // });

      // let foundPixels = 0;

      image.forEach((yArr, y) => {
        yArr.forEach((xArr, x) => {
          // console.log(y, x);
          // loop over all drawn objects
          this.boxes.forEach((b) => {
            // const classID = b.classId;
            if (
              b.x <= x &&
              x <= b.x + b.width &&
              b.y <= y &&
              y <= b.y + b.height
            ) {
              // console.log(`pixel ${y},${x} contains object ID ${classID}`);
              // foundPixels += 1;
              const xGrid = Math.ceil((x + 1) / 64);
              const yGrid = Math.ceil((y + 1) / 64);
              arr[yGrid - 1][xGrid - 1][b.classId] += 1;
            }
          });
        });
      });

      console.log("done", arr);

      const res = await (
        await axios.post(
          `http://localhost:9191/SearchClassMilvus?topk=500`,
          arr
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

    selectColor(colorNum, colors) {
      if (colors < 1) colors = 1; // defaults to one color - avoid divide by zero
      return "hsl(" + ((colorNum * (360 / colors)) % 360) + ",100%,50%)";
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
      console.log("start painting");
      const mousePos = this.getMousePos(e);

      this.painting = true;
      this.drawingBox.x = mousePos.x;
      this.drawingBox.y = mousePos.y;
    },
    finishedPainting() {
      this.painting = false;
      this.drawingBox.classId = this.selectedClass;
      const overlayCanvas = this.$refs.overlayCanvas;
      // overlayCanvas.getContext("2d").strokeStyle = "blue";
      // overlayCanvas.getContext("2d").lineWidth = 3;
      console.log(this.colors[this.selectedClass]);
      overlayCanvas.getContext("2d").fillStyle = this.colors[
        this.selectedClass
      ];

      console.log("finished painting", this.drawingBox);
      this.boxes.push(this.drawingBox);

      overlayCanvas
        .getContext("2d")
        .fillRect(
          this.drawingBox.x,
          this.drawingBox.y,
          this.drawingBox.width,
          this.drawingBox.height
        );
    },
    draw(e) {
      if (!this.painting) return;
      console.log("draw");

      const canvas = this.$refs.canvas;

      const mousePos = this.getMousePos(e);
      this.drawingBox.width = mousePos.x - this.drawingBox.x;
      this.drawingBox.height = mousePos.y - this.drawingBox.y;

      canvas.getContext("2d").clearRect(0, 0, canvas.width, canvas.height);
      canvas
        .getContext("2d")
        .strokeRect(
          this.drawingBox.x,
          this.drawingBox.y,
          this.drawingBox.width,
          this.drawingBox.height
        );
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
    position: relative;
    margin-top: 1rem;
    min-height: 0;

    canvas {
      border: lightgray thin solid;
      min-height: 0;
      max-height: 100%;
      max-width: 100%;
    }

    .overlay-canvas {
      position: absolute;
      top: 0;
      left: 0;
      z-index: -1;
    }
  }
}
</style>
