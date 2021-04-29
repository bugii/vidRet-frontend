<template>
  <div class="video-modal">
    <div class="top-bar">
      <CloseIcon @click.native="close" class="close-icon" />
    </div>
    <h1>{{ videoNr }}</h1>
    <div class="video-container">
      <video ref="video" autoplay controls>
        <source :src="videoPath" type="video/mp4" />
      </video>
    </div>
    <button @click.prevent="setTimestamp">
      Set timestamp
    </button>

    <div>
      <div>selected Timestamp: {{ this.timestamp }}</div>
    </div>

    <button @click.prevent="submit">
      Submit
    </button>
  </div>
</template>

<script>
import axios from "axios";

import CloseIcon from "@/components/Icons/CloseIcon.vue";

export default {
  components: {
    CloseIcon,
  },

  props: ["videoNr", "imgName"],

  data() {
    return {
      timestamp: null,
    };
  },

  methods: {
    async login() {
      const res = await (
        await axios.post("http://localhost:8080/api/login", {
          username: "admin",
          password: "password",
        })
      ).data;

      console.log(res);
      this.sessionId = res.sessionId;
    },

    async submitToAPI() {
      const res = await (
        await axios.get("http://localhost:8080/submit", {
          params: {
            item: `${this.$props.videoNr}`, // item -  item which is to be submitted
            timecode: `${this.timestamp}:00`, // timecode - in this case, we use the timestamp in the form HH:MM:SS:FF
            sessionId: this.sessionId, // the sessionId, as always
          },
        })
      ).data;

      console.log(res);
    },

    close() {
      console.log("click");
      this.$emit("closeVideoModal");
    },

    setTimestamp() {
      this.timestamp = new Date(this.$refs.video.currentTime * 1000)
        .toISOString()
        .substr(11, 8);
    },

    async submit() {
      //TODO: Move login into nav bar or so
      await this.login();
      await this.submitToAPI();
    },
  },

  computed: {
    videoPath() {
      return `http://localhost:9191/videos/${this.$props.videoNr}/${this.$props.videoNr}.mp4`;
    },
  },

  async mounted() {
    // jump to timestamp from selected keyframe

    const res = await (
      await axios.post(
        `http://localhost:9191/getPictureInformation?pictureId=${this.imgName}`
      )
    ).data;

    const startTime = res.startTime;

    this.$refs.video.currentTime = Math.floor(startTime);

    console.log(res);
  },
};
</script>

<style lang="scss" scoped>
.video-modal {
  position: fixed;
  background: rgba(255, 255, 255, 0.975);

  width: 100%;
  height: 100vh;
  top: 0;
  right: 0;

  .top-bar {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    height: 2rem;

    .close-icon {
      height: 100%;
      padding: 0.5rem;
      cursor: pointer;
    }
  }

  .video-container {
    width: 100%;
    video {
      width: 100%;
    }
  }
}
</style>
