<template>
  <div class="tile">
    <audio autoPlay playsInline ref="audio">
      <track kind="captions" />
    </audio>

    <video v-show="videoTrack" autoPlay muted playsInline ref="video"></video>
    <p v-show="videoTrack" class="participant-name">{{ username }}</p>

    <no-video-tile v-show="!videoTrack" :username="username"></no-video-tile>

    <template v-if="isLocal">
      <controls
        :handleVideoClick="handleVideoClick"
        :handleAudioClick="handleAudioClick"
        :handleScreenshareClick="handleScreenshareClick"
        :audioTrack="audioTrack"
        :videoTrack="videoTrack"
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
    "username",
    "videoTrack",
    "audioTrack",
    "isLocal",
    "handleVideoClick",
    "handleAudioClick",
    "handleScreenshareClick",
    "leaveCall",
    "disableScreenShare",
  ],
  mounted() {
    // eslint-disable-next-line no-debugger
    // this.handleVideo(this.videoTrack);
    this.handleAudio(this.audioTrack);
  },
  updated() {
    console.log("VUE UPDATED");
    // this.username = this.participant?.user_name;
    // For later optimization, this can be done more selectively
    // using "track-started" and "track-stopped" events.
    // this.handleVideo(this.videoTrack);
    // this.handleAudio(this.audioTrack);
  },
  methods: {
    // Add srcObject to video element
    handleVideo(track) {
      console.log("HANDLE VIDEO");
      if (!track) return;
      if (!this.$refs.video.srcObject) {
        this.$refs.video.srcObject = new MediaStream();
      }
      console.log("this.$refs.video.srcObject", this.$refs.video.srcObject);
      const existingStream = this.$refs.video.srcObject;
      const existingTracks = existingStream?.getVideoTracks() ?? [];
      this.refreshTrack(existingStream, existingTracks, track);
    },
    // Add srcObject to audio element
    handleAudio(track) {
      // If the participant is local or has their audio off,
      // early out.
      if (this.isLocal || !track) return;
      if (!this.$refs.audio.srcObject) {
        this.$refs.audio.srcObject = new MediaStream();
      }
      const existingStream = this.$refs.audio.srcObject;
      const existingTracks = existingStream.getAudioTracks() ?? [];
      this.refreshTrack(existingStream, existingTracks, track);
    },
    refreshTrack(existingStream, oldTracks, newTrack) {

      const trackCount = oldTracks.length;
      // If there is no matching old track,
      // just add the new track.
      if (trackCount === 0) {
        existingStream.addTrack(newTrack);
        return;
      }
      if (trackCount > 1) {
        console.warn(
          `expected up to 1 media track, but got ${trackCount}. Only using the first one.`
        );
      }
      const oldTrack = oldTracks[0];
      // If the IDs of the old and new track don't match,
      // replace the old track with the new one.
      if (oldTrack.id !== newTrack.id) {
        existingStream.removeTrack(oldTrack);
        existingStream.addTrack(newTrack);
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
