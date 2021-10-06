<template>
  <!-- <template v-if="p."> </template> -->
  <div>
    <template v-if="video">
      <video autoPlay muted playsInline :srcObject="videoSource"></video>
    </template>
    <controls
      :handleVideoClick="handleVideoClick"
      :handleAudioClick="handleAudioClick"
      :participant="participant"
    />
  </div>
</template>

<script>
import Controls from "./Controls.vue";

export default {
  name: "Tile",
  components: {
    Controls,
  },
  props: ["participant", "handleVideoClick", "handleAudioClick"],
  data() {
    return {
      videoSource: null,
    };
  },
  computed: {
    video() {
      return this.participant.video;
    },
  },
  mounted() {
    this.handleVideo(this.participant);
  },
  watch: {
    particpant: {
      deep: true,
      handler: function(newVal, oldVal) {
        console.log("Prop changed: ", newVal, " | was: ", oldVal);
        this.handleVideo(newVal);
      },
    },
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

<style scoped></style>
