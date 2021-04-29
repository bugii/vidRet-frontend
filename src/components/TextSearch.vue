<template>
  <div class="text-search-container">
    <p class="box-title">
      Text-based search
    </p>
    <input
      v-model="keyword"
      type="text"
      id="textsearch"
      name="textsearch"
      placeholder="Enter keyword ..."
    />

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
      keyword: "",
    };
  },
  methods: {
    async search() {
      console.log("searching for keyword", this.keyword);
      this.$emit("fetchStart");
      // Faking api call
      // setTimeout(() => {
      //   this.$emit("fetchEnd", [
      //     "shot00032_1.png",
      //     "shot00032_2.png",
      //     "shot00032_3.png",
      //     "shot00032_4.png",
      //     "shot00032_5.png",
      //     "shot00032_6.png",
      //     "shot00032_7.png",
      //     "shot00032_8.png",
      //     "shot00032_9.png",
      //     "shot00032_10.png",
      //     "shot00032_11.png",
      //     "shot00032_12.png",
      //     "shot00032_13.png",
      //     "shot00032_14.png",
      //     "shot00032_15.png",
      //     "shot00032_16.png",
      //     "shot00032_17.png",
      //     "shot00032_18.png",
      //     "shot00032_19.png",
      //     "shot00032_20.png",
      //     "shot00032_21.png",
      //     "shot00032_22.png",
      //     "shot00032_23.png",
      //     "shot00032_24.png",
      //     "shot00032_25.png",
      //     "shot00032_26.png",
      //     "shot00032_27.png",
      //     "shot00032_28.png",
      //     "shot00032_29.png",
      //     "shot00032_30.png",
      //     "shot00032_31.png",
      //     "shot00032_32.png",
      //   ]);
      // }, 500);

      try {
        const res = await (
          await axios.post(
            `http://localhost:9191/SearchForText?searchText=${this.keyword}`
          )
        ).data;

        const arr = res.map((e) => e.pictureId);
        console.log(arr);

        this.$emit("fetchEnd", arr);

        // const res2 = await (
        //         await axios.post(
        //           `http://localhost:9191/getPictureInformation?pictureId=${res}.png`
        //         )
        //       ).data;

        //       console.log(res2);
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.text-search-container {
  display: flex;
  flex-direction: column;

  .box-title {
    margin-bottom: 1rem;
  }

  button {
    margin-top: 1rem;
  }
}
</style>
