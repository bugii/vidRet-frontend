<template>
  <div>
    <p class="box-title">
      Combined search
    </p>

    <div class="slidecontainer">
      class: {{ classFraction }}
      <input
        @input="$emit('update', $event.target.value)"
        type="range"
        min="0"
        max="1"
        step="0.01"
        :value="classFraction"
        class="slider"
      />
      color: {{ Math.round((1 - classFraction + Number.EPSILON) * 100) / 100 }}
    </div>
    <button @click.prevent="$emit('search')">
      Search
    </button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  props: ["classFraction"],

  methods: {
    async search() {
      console.log("searching for the objects", this.boxes);
      this.$emit("fetchStart");

      const res = await (
        await axios.post(`http://localhost:9191/SearchCombinedMilvus?topk=500`)
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
  },
};
</script>

<style lang="scss" scoped>
.box-title {
  font-weight: 600;
}

.slidecontainer {
  width: 100%; /* Width of the outside container */
  display: flex;
}

/* The slider itself */
.slider {
  -webkit-appearance: none; /* Override default CSS styles */
  appearance: none;
  width: 100%; /* Full-width */
  height: 25px; /* Specified height */
  background: #d3d3d3; /* Grey background */
  outline: none; /* Remove outline */
  opacity: 0.7; /* Set transparency (for mouse-over effects on hover) */
  -webkit-transition: 0.2s; /* 0.2 seconds transition on hover */
  transition: opacity 0.2s;
}

/* Mouse-over effects */
.slider:hover {
  opacity: 1; /* Fully shown on mouse-over */
}

/* The slider handle (use -webkit- (Chrome, Opera, Safari, Edge) and -moz- (Firefox) to override default look) */
.slider::-webkit-slider-thumb {
  -webkit-appearance: none; /* Override default look */
  appearance: none;
  width: 25px; /* Set a specific slider handle width */
  height: 25px; /* Slider handle height */
  background: #04aa6d; /* Green background */
  cursor: pointer; /* Cursor on hover */
}

.slider::-moz-range-thumb {
  width: 25px; /* Set a specific slider handle width */
  height: 25px; /* Slider handle height */
  background: #04aa6d; /* Green background */
  cursor: pointer; /* Cursor on hover */
}

button {
  display: flex;
  width: 100%;
}
</style>
