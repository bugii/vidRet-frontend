<template>
  <div style="height: 100%;">
    <LoadingSpinner v-if="isLoadingResults" />
    <VideoModal
      v-if="showModal"
      @closeVideoModal="closeModal"
      :videoNr="videoNr"
      :imgName="imgName"
      :sessionId="sessionId"
    />
    <div class="results-container">
      <div
        class="video-container"
        v-for="e in resultThumbnailsPerVideo"
        :key="e.video"
      >
        <img
          class="result-image"
          v-for="img in e.results"
          :key="img.imgPath"
          :src="img.imgPath"
          @click="openVideoPlayer(img.videoNr, img.imgName)"
          alt="thumbnail"
        />
      </div>
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

  props: ["isLoadingResults", "results", "sessionId"],

  computed: {
    resultThumbnailsPerVideo() {
      const results = this.$props.results;

      const thumbsPerVideo = {};

      console.log("results", results);
      // Take all the returned image names and create local paths from it

      results.forEach((imgName) => {
        const videoNr = imgName.slice(4, 9);
        const imgPath = `http://localhost:9191/thumbnails/${videoNr}/${imgName.replace(
          "_RKF",
          ""
        )}`;

        const current = thumbsPerVideo[videoNr];
        if (current) {
          current.results.push({ imgPath, videoNr, imgName });
        } else {
          thumbsPerVideo[videoNr] = {
            video: videoNr,
            results: [{ imgPath, videoNr, imgName }],
          };
        }
      });

      console.log(thumbsPerVideo);
      return thumbsPerVideo;
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
  height: 100%;

  .video-container {
    padding: 0.25rem;
    margin: 0.25rem;
    border: gainsboro 1px solid;
    border-radius: 0.25rem;
    width: 100%;
  }

  .result-image {
    width: 10%;
    min-height: 4rem;
    min-width: 8rem;
    object-fit: contain;
  }
}
</style>
