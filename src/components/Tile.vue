<template>
  <!-- <template v-if="p."> </template> -->
  <div class="tile">
    <template v-if="participant.video">
      <video autoPlay muted playsInline :srcObject="videoSource"></video>
    </template>
    <template v-else>
      <no-video-tile></no-video-tile>
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
  name: "Tile",
  components: {
    Controls,
    NoVideoTile,
  },
  props: ["participant", "handleVideoClick", "handleAudioClick", "leaveCall"],
  data() {
    return {
      videoSource: null,
    };
  },
  mounted() {
    this.handleVideo(this.participant);
  },
  updated() {
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
}
video {
  width: 100%;
  border-radius: 16px;
}
</style>
