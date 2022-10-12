<template>
  <div class="tile">
    <audio autoplay playsInline ref="audioRef">
      <track kind="captions" />
    </audio>

    <template v-if="participant.video">
      <video autoplay muted playsInline ref="videoRef"></video>
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

    this.$refs.videoRef?.addEventListener("canplay", this.handleVideoCanPlay);
    this.$refs.audioRef?.addEventListener("canplay", this.handleAudioCanPlay);
    navigator.mediaDevices.addEventListener("devicechange", this.playTracks);
  },
  unmounted() {
    this.$refs.videoRef?.removeEventListener("canplay", this.handleVideoCanPlay);
    this.$refs.audioRef?.removeEventListener("canplay", this.handleAudioCanPlay);
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
    handleVideoCanPlay () {
      if (!this.$refs.videoRef.paused) return;
      this.$refs.videoRef.play().catch((e) => {
        if (e instanceof DOMException && e.name === 'NotAllowedError') {
          console.error("Autoplay failed", e);
        } else console.error("Error playing video", e);
      });
    },
    handleAudioCanPlay () {
      if (!this.$refs.audioRef.paused) return;
      this.$refs.audioRef.play().catch((e) => {
        if (e instanceof DOMException && e.name === 'NotAllowedError') {
          console.error("Autoplay failed", e);
        } else console.error("Error playing audio: ", e);
      });
    },
    playTracks () {
      this.handleVideoCanPlay();
      this.handleAudioCanPlay();
    },

    // Add srcObject to video element
    handleVideo() {
      const p = this.participant;

      // If the participant has their video off,
      // early out.
      if (!p?.video) return;

      try {
        const track = p?.tracks?.video?.persistentTrack;
        this.$refs.videoRef.srcObject = new MediaStream([track]);
        this.$refs.videoRef.load();
      } catch (e) {
        console.error('show up a modal to get user gesture', e);
      }
    },

    // Add srcObject to audio element
    handleAudio() {
      const p = this.participant;
      // If the participant is local or has their audio off,
      // early out.
      if (!p || p.local || !p.audio) return;

      try {
        const track = p?.tracks?.audio?.persistentTrack;
        this.$refs.audioRef.srcObject = new MediaStream([track]);
      } catch (e) {
        console.error('show up a modal to get user gesture', e);
      }
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
audio {
  position: absolute;
  visibility: hidden;
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
