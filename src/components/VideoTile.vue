<template>
  <div class="tile">
    <audio autoPlay playsInline ref="audioRef">
      <track kind="captions" />
    </audio>

    <template v-if="participant.video">
      <video autoPlay muted playsInline ref="videoRef"></video>
      <p class="participant-name">{{ username }}</p>
    </template>

    <template v-else>
      <no-video-tile :username="username"></no-video-tile>
    </template>

    <template v-if="participant.local">
      <call-controls
        :handle-video-click="handleVideoClick"
        :handle-audio-click="handleAudioClick"
        :handle-screenshare-click="handleScreenshareClick"
        :participant="participant"
        :leave-call="leaveCall"
        :disable-screen-share="disableScreenShare"
      />
    </template>
  </div>
</template>

<script>
import CallControls from "./CallControls.vue";
import NoVideoTile from "./NoVideoTile.vue";

export default {
  name: "VideoTile",
  components: {
    CallControls,
    NoVideoTile,
  },
  props: [
    "participant",
    "handleVideoClick",
    "handleAudioClick",
    "handleScreenshareClick",
    "leaveCall",
    "disableScreenShare",
  ],
  data() {
    return {
      username: "Guest",
    };
  },
  mounted() {
    this.username = this.participant?.user_name;
    this.handleVideo(this.participant);
    this.handleAudio(this.participant);

    navigator.mediaDevices.addEventListener("devicechange", this.playTracks);
  },
  unmounted() {
    navigator.mediaDevices.removeEventListener("devicechange", this.playTracks);
  },
  updated() {
    this.username = this.participant?.user_name;
    // For later optimization, this can be done more selectively
    // using "track-started" and "track-stopped" events.
    this.handleVideo(this.participant);
    this.handleAudio(this.participant);
  },
  methods: {
    playTracks () {
      this.$refs.videoRef.play();
      this.$refs.audioRef.play();
    },

    // Add srcObject to video element
    handleVideo() {
      const p = this.participant;

      // If the participant has their video off,
      // early out.
      if (!p?.video) return;

      const track = p?.tracks?.video?.persistentTrack;
      this.$refs.videoRef.srcObject = new MediaStream([track]);
      this.$refs.videoRef.play();
    },

    // Add srcObject to audio element
    handleAudio() {
      const p = this.participant;
      // If the participant is local or has their audio off,
      // early out.
      if (!p || p.local || !p.audio) return;

      const track = p?.tracks?.audio?.persistentTrack;
      this.$refs.audioRef.srcObject = new MediaStream([track]);
      this.$refs.audioRef.play();
    },
  },
};
</script>

<style scoped>
.tile {
  max-width: 50%;
  flex: 1 1 350px;
  margin: 10px 20px;
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
    max-width: 100%;
    margin: 10px 20px;
  }
}
</style>
