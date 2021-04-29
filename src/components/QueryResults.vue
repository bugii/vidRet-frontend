<template>
  <div>
    <LoadingSpinner v-if="isLoadingResults" />
    <VideoModal
      v-if="showModal"
      @closeVideoModal="closeModal"
      :videoNr="videoNr"
      :imgName="imgName"
    />
    <div class="results-container">
      <img
        class="result-image"
        v-for="img in resultThumbnails"
        :key="img.imgPath"
        :src="img.imgPath"
        @click="openVideoPlayer(img.videoNr, img.imgName)"
        alt="thumbnail"
      />
    </div>
  </div>
</template>

<script>
import LoadingSpinner from "@/components/LoadingSpinner.vue";
import VideoModal from "@/components/VideoModal.vue";

export default {
  components: {
    LoadingSpinner,
    VideoModal,
  },

  data() {
    return { showModal: false, videoNr: null, imgName: null };
  },

  props: ["isLoadingResults", "results"],

  computed: {
    resultThumbnails() {
      const results = this.$props.results;
      console.log("results", results);
      // Take all the returned image names and create local paths from it

      const updatedArr = results.map((imgName) => {
        const videoNr = imgName.slice(4, 9);
        const imgPath = `http://localhost:9191/thumbnails/${videoNr}/${imgName.replace(
          "_RKF",
          ""
        )}`;

        return { imgPath, videoNr, imgName };
      });

      return updatedArr;
    },
  },

  methods: {
    openVideoPlayer(videoNr, imgName) {
      console.log("open video player for video", videoNr);

      this.videoNr = videoNr;
      this.imgName = imgName;

      this.showModal = true;
    },
    closeModal() {
      console.log("close modal");
      this.showModal = false;
    },
  },
};
</script>

<style lang="scss" scoped>
.results-container {
  display: flex;
  flex-wrap: wrap;
  padding: 1rem;
  overflow: auto;

  .result-image {
    width: 10%;
    min-height: 4rem;
    min-width: 8rem;
    object-fit: contain;
  }
}
</style>
