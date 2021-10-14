<template>
  <div class="screenshare-tile">
    <video autoPlay muted playsInline :srcObject="videoSource"></video>
  </div>
</template>

<script>
export default {
  name: "ScreenshareTile",
  props: ["participant"],
  data() {
    return {
      videoSource: null,
    };
  },
  mounted() {
    console.log(this.participant.screenVideoTrack);
    this.handleVideo(this.participant);
  },
  updated() {
    this.handleVideo(this.participant);
  },
  methods: {
    handleVideo() {
      console.log(this.participant);
      if (!this.participant?.screen) return;
      const videoTrack = this.participant?.screenVideoTrack;
      console.log("HEYYYYY");
      console.log(videoTrack);
      const source = new MediaStream([videoTrack]);
      this.videoSource = source;
    },
  },
};
</script>

<style>
.screenshare-tile {
  flex: 1 1 90%;
  margin: 10px 20px;
  position: relative;
}
</style>
