<template>
  <div class="tile">
    <audio autoPlay playsInline :srcObject="audioSource">
      <track kind="captions" />
    </audio>

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
        :handleScreenshareClick="handleScreenshareClick"
        :participant="participant"
        :leaveCall="leaveCall"
        :disableScreenShare="disableScreenShare"
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
      videoSource: null,
      audioSource: null,
      username: "Guest",
    };
  },
  mounted() {
    this.username = this.participant?.user_name;
    this.handleVideo(this.participant);
    this.handleAudio(this.participant);
  },
  updated() {
    this.username = this.participant?.user_name;

    if (!this.videoSource) {
      this.handleVideo(this.participant);
    }
    if (!this.audioSource) {
      this.handleAudio(this.participant);
    }
  },
  methods: {
    // Add srcObject to video element
    handleVideo() {
      if (!this.participant?.video) return;
      const videoTrack = this.participant?.tracks?.video?.persistentTrack;
      const source = new MediaStream([videoTrack]);
      this.videoSource = source;
    },
    // Add srcObject to audio element
    handleAudio() {
      if (this.participant?.local) return;
      if (!this.participant?.tracks?.audio?.persistentTrack) return;
      const audioTrack = this.participant?.tracks?.audio?.persistentTrack;
      const source = new MediaStream([audioTrack]);
      this.audioSource = source;
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
