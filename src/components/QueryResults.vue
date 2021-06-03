<template>
  <div style="height: 100%;">
    <LoadingSpinner v-if="isLoadingResults" />
    <VideoModal
      v-if="showModal"
      @closeVideoModal="closeModal"
      :videoNr="videoNr"
      :imgName="imgName"
      :sessionId="sessionId"
      :startTime="startTime"
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
          @click.shift="submit(img)"
          @click.exact="
            openVideoPlayer(img.videoNr, img.imgName, img.startTime)
          "
          alt="thumbnail"
        />
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import LoadingSpinner from "@/components/LoadingSpinner.vue";
import VideoModal from "@/components/VideoModal.vue";

export default {
  components: {
    LoadingSpinner,
    VideoModal,
  },

  data() {
    return { showModal: false, videoNr: null, imgName: null, startTime: null };
  },

  props: ["isLoadingResults", "results", "sessionId"],

  computed: {
    resultThumbnailsPerVideo() {
      const results = this.results;

      const thumbsPerVideo = {};

      // console.log("results", results);
      // Take all the returned image names and create local paths from it

      results.forEach((img) => {
        const videoNr = img.pictureId.slice(4, 9);
        const imgPath = `http://localhost:9191/thumbnails/${videoNr}/${img.pictureId.replace(
          "_RKF",
          ""
        )}`;

        const current = thumbsPerVideo[videoNr];
        if (current) {
          current.results.push({
            imgPath,
            videoNr,
            imgName: img.pictureId,
            frame: Math.floor((img.endFrame + img.startFrame) / 2),
            distance: img.distance,
            startTime: img.startTime,
          });
        } else {
          thumbsPerVideo[videoNr] = {
            video: videoNr,
            results: [
              {
                imgPath,
                videoNr,
                imgName: img.pictureId,
                frame: Math.floor((img.endFrame + img.startFrame) / 2),
                distance: img.distance,
                startTime: img.startTime,
              },
            ],
          };
        }
      });

      // console.log("thumb per video", thumbsPerVideo);
      const sortedFramesPerVideo = Object.entries(thumbsPerVideo).map(
        ([k, v]) => {
          // console.log("results for video ", k, v);
          return {
            video: k,
            results: v.results.sort((a, b) => b.distance - a.distance),
          };
        }
      );
      // console.log(sortedFramesPerVideo);

      const completelySortedFrames = sortedFramesPerVideo.sort(
        (a, b) => b.results[0].distance - a.results[0].distance
      );

      return completelySortedFrames;
    },
  },

  methods: {
    async submit(img) {
      console.log(img);
      try {
        const res = await (
          await axios.get("https://interactivevideoretrieval.com/submit", {
            params: {
              item: img.videoNr, // item -  item which is to be submitted
              frame: img.frame, // timecode - in this case, we use the timestamp in the form HH:MM:SS:FF
              session: this.sessionId, // the sessionId, as always
            },
          })
        ).data;

        console.log(res);
        if (res.submission === "CORRECT") {
          this.$toast.success(res.description);
        } else if (res.submission === "WRONG") {
          this.$toast.error(res.description);
        } else {
          this.$toast(res.description);
        }
      } catch (error) {
        this.$toast.error(error.message);
      }
    },
    openVideoPlayer(videoNr, imgName, startTime) {
      console.log("open video player for video", videoNr, "at", startTime);

      this.videoNr = videoNr;
      this.imgName = imgName;
      this.startTime = startTime;

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
  flex-direction: column;
  // flex-wrap: wrap;
  padding: 1rem;
  overflow: auto;
  height: 100%;

  .video-container {
    display: flex;
    flex-wrap: wrap;
    // align-self: flex-start;
    padding: 0.25rem;
    margin: 0.25rem;
    border: gainsboro 1px solid;
    border-radius: 0.25rem;
    // width: 100%;
  }

  .result-image {
    display: block;
    width: 10%;
    min-height: 4rem;
    min-width: 8rem;
    object-fit: contain;
  }
}
</style>
