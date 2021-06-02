<template>
  <div id="app">
    <div v-if="!loggedIn" class="not-logged-in-info">
      You are currently NOT logged in. Please do so before submitting!
    </div>

    <div class="left">
      <div class="search-results">
        <label for="searchResults">Results</label>
        <input
          class="number-search-results"
          type="number"
          v-model="searchResults"
        />
      </div>

      <div v-if="!loggedIn" class="submission-credentials">
        <div class="box-title">Submission credentials</div>
        <label for="username">User name </label>
        <input v-model="username" type="text" name="username" id="username" />
        <label for="password">Password </label>
        <input
          v-model="password"
          type="password"
          name="password"
          id="password"
        />
        <button class="submission-button" @click="login">Log in</button>
      </div>

      <TextSearch @search="textSearch" @update="updateText" :keyword="text" />
      <ClassSearch
        :boxes="classBoxes"
        @search="classSearch"
        @addBox="addClassBox"
        @clear="clearClassBoxes"
      />
      <ColorSearch
        :colors="colorsRGB"
        @setColor="setColor"
        @clear="clearColors"
        @search="colorSearch"
      />
      <CombinedSearch
        @search="combinedSearch"
        :classFraction="classFraction"
        @update="setClassFraction"
      />
    </div>
    <div class="right">
      <QueryResults
        :sessionId="sessionId"
        :results="results"
        :isLoadingResults="isLoadingResults"
      />
    </div>
  </div>
</template>

<script>
import axios from "axios";

import TextSearch from "@/components/TextSearch.vue";
import ClassSearch from "@/components/ClassSearch.vue";
import ColorSearch from "@/components/ColorSearch.vue";
import QueryResults from "@/components/QueryResults.vue";
import CombinedSearch from "@/components/CombinedSearch.vue";

export default {
  data() {
    return {
      username: null,
      password: null,
      sessionId: null,
      searchResults: 100,
      isLoadingResults: false,
      text: "",
      classBoxes: [],
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
      classFraction: 0.5,
      results: [],
    };
  },

  components: {
    TextSearch,
    ClassSearch,
    ColorSearch,
    QueryResults,
    CombinedSearch,
  },

  computed: {
    colorsLAB() {
      return this.colorsRGB.map((r) => r.map((c) => this.rgb2lab(c)));
    },
    loggedIn() {
      if (this.sessionId !== null) return true;
      return false;
    },
  },

  methods: {
    async login() {
      // console.log("logging in with", this.username, this.password);

      const res = await (
        await axios.post(
          "https://test.interactivevideoretrieval.com/api/login",
          {
            username: this.username,
            password: this.password,
          }
        )
      ).data;

      console.log(res);
      this.sessionId = res.sessionId;
    },

    updateText(val) {
      this.text = val;
    },

    async textSearch() {
      console.log("searching for keyword", this.text);
      this.isLoadingResults = true;

      try {
        const res = await (
          await axios.post(
            `http://localhost:9191/SearchForText?searchText=${this.text}`
          )
        ).data;

        // const arr = res.map((e) => e.pictureId);
        // console.log(arr);
        console.log("result", res);
        this.fetchEnd(res);
      } catch (error) {
        console.log(error);
      }
    },

    async classSearch() {
      console.log("searching for the objects", this.classBoxes);
      this.isLoadingResults = true;

      const arr = this.getQueryArray();

      const res = await (
        await axios.post(
          `http://localhost:9191/SearchClassMilvus?topk=${this.searchResults}`,
          arr
        )
      ).data;

      console.log("result", res);

      // const pictureIds = res.map((e) => e.pictureId);

      this.fetchEnd(res);
    },

    async colorSearch() {
      console.log(
        "searching for colors",

        this.colorsLAB
      );

      this.isLoadingResults = true;

      const res = await (
        await axios.post(
          `http://localhost:9191/SearchColorMilvus?topk=${this.searchResults}`,
          this.colorsLAB
        )
      ).data;

      console.log("result", res);

      // const pictureIds = res.map((e) => e.pictureId);

      this.fetchEnd(res);
    },

    async combinedSearch() {
      this.isLoadingResults = true;

      const classArray = this.getQueryArray();
      const colors = this.colorsLAB;

      console.log("searching for the objects", this.classBoxes);

      console.log(
        "searching for colors",

        colors
      );

      const res = await (
        await axios.post(
          `http://localhost:9191/SearchCombinedMilvus?topk=${this.searchResults}&classFraction=${this.classFraction}`,
          [classArray, colors]
        )
      ).data;

      console.log("result", res);

      // const pictureIds = res.map((e) => e.pictureId);

      this.fetchEnd(res);
    },

    fetchEnd(results) {
      this.isLoadingResults = false;
      this.results = results;
    },

    addClassBox(val) {
      this.classBoxes.push(val);
    },

    clearClassBoxes() {
      this.classBoxes = [];
    },

    setColor(y, x, rgbColor) {
      this.$set(this.colorsRGB[y], x, rgbColor);
    },

    clearColors() {
      this.colorsRGB = [
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
      ];
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

    getQueryArray() {
      // convert to 2D array
      const arr = [...Array(8)].map(() => Array(8).fill(Array(81).fill(0)));
      const image = [...Array(512)].map(() => Array(512).fill(0));

      image.forEach((yArr, y) => {
        yArr.forEach((xArr, x) => {
          // loop over all drawn objects
          this.classBoxes.forEach((b) => {
            // const classID = b.classId;
            if (
              b.x <= x &&
              x <= b.x + b.width &&
              b.y <= y &&
              y <= b.y + b.height
            ) {
              const xGrid = Math.ceil((x + 1) / 64);
              const yGrid = Math.ceil((y + 1) / 64);
              arr[yGrid - 1][xGrid - 1][b.classId] += 1;
            }
          });
        });
      });
      return arr;
    },

    setClassFraction(val) {
      this.classFraction = val;
      // console.log(val);
    },
  },
};
</script>

<style lang="scss">
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  font: inherit;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  display: flex;
  margin: 1rem;
  height: calc(100vh - 2rem);

  .not-logged-in-info {
    position: fixed;
    z-index: 1;
    top: 0;
    left: 0;
    width: 100vw;
    padding: 0.5rem;
    background: red;
    color: white;
    display: flex;
    justify-content: center;
  }

  .box-title {
    font-weight: 600;
  }

  .submission-button {
    display: block;
    width: 100%;
    margin-top: 0.25rem;
  }

  input,
  label {
    width: 100%;
  }

  .left {
    flex: 0 0 20%;
    display: flex;
    flex-direction: column;
    overflow: auto;

    .search-results {
      display: flex;

      .number-search-results {
        margin-left: 1rem;
        width: 8ch;
      }
    }

    .search-box {
      margin-top: 1rem;
    }
    // .search-box-1 {
    //   flex-basis: content;
    // }
    // .search-box-2,
    // .search-box-3,
    // .search-box-4 {
    //   flex-grow: 1;
    // }
  }

  .right {
    height: 100%;
    position: relative;
  }
}
</style>
