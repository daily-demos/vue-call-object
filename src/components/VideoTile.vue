<template>
  <div class="tile">
    <template v-if="participant.video">
      <video autoPlay muted playsInline :srcObject="videoSource"></video>
      <p class="participant-name">{{ username }}</p>
    </template>

    <template v-else>
      <no-video-tile :username="username"></no-video-tile>
    </template>

    <template v-if="participant.local">
      <controls
        :handleVideoClick="handleVideoClick"
        :handleAudioClick="handleAudioClick"
        :participant="participant"
        :leaveCall="leaveCall"
      />
    </template>
  </div>
</template>

<script>
import Controls from "./Controls.vue";
import NoVideoTile from "./NoVideoTile.vue";

export default {
  name: "VideoTile",
  components: {
    Controls,
    NoVideoTile,
  },
  props: ["participant", "handleVideoClick", "handleAudioClick", "leaveCall"],
  data() {
    return {
      videoSource: null,
      username: "Guest",
    };
  },
  mounted() {
    this.username = this.participant?.user_name;
    this.handleVideo(this.participant);
  },
  updated() {
    this.username = this.participant?.user_name;

    if (this.videoSource) return;
    this.handleVideo(this.participant);
  },
  methods: {
    handleVideo() {
      if (!this.participant?.video) return;
      const videoTrack = this.participant?.tracks?.video?.persistentTrack;
      const source = new MediaStream([videoTrack]);
      this.videoSource = source;
    },
  },
};
</script>

<style scoped>
.tile {
  max-width: 480px;
  min-width: 300px;
  position: relative;
  margin: 10px 20px;
  flex: 1;
  position: relative;
}
video {
  width: 100%;
  border-radius: 16px;
}
.participant-name {
  position: absolute;
  color: #fff;
  top: 12px;
  right: 12px;
  margin: 0;
}

@media screen and (max-width: 700px) {
  .tile {
    width: 100%;
    margin: 10px 60px;
  }
}
</style>
