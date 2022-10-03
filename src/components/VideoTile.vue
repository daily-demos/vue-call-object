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
    this.handleVideo(this.videoTrack);
    this.handleAudio(this.audioTrack);
  },
  updated() {
    console.log("VUE UPDATED");
    // this.username = this.participant?.user_name;
    // // For later optimization, this can be done more selectively
    // // using "track-started" and "track-stopped" events.
    // this.handleVideo(this.participant);
    // this.handleAudio(this.participant);
  },
  methods: {
    // Add srcObject to video element
    handleVideo(track) {
      const newStream = this.updateSource(this.$refs.video.srcObject, track);
      if (newStream) {
        this.$refs.video.srcObject = newStream;
      }
    },

    // Add srcObject to audio element
    handleAudio(audioTrack) {
      // If the participant is local or has their audio off,
      // early out.
      if (this.isLocal || !audioTrack) return;

      const newStream = this.updateSource(this.$refs.audio.srcObject, audioTrack);
      if (newStream) {
        this.$refs.audio.srcObject = newStream;
      }
    },

    // updateSource() updates the given stream with new tracks OR
    // returns an entirely new stream to the caller.
    updateSource(stream, newTrack) {
      const existingTracks = stream?.getTracks();
      console.log("newTrack: ", newTrack)
      // If the stream parameter contains no existing tracks,
      // just return a new MediaStream to set. This should
      // only happen the first time the tile is initialized.
      if (!existingTracks || existingTracks.length === 0) {
        return new MediaStream([newTrack]);
      }
      if (existingTracks.length > 1) {
        console.warn(`expected 1 track, found ${existingTracks.length}. Only using the first one.`);
      }
      const existingTrack = existingTracks[0];
      // If existing track is different from the new track,
      // remove the existing track and add the new one.
      if (newTrack.id !== existingTrack.id) {
        stream.removeTrack(existingTrack);
        stream.addTrack(newTrack);
      }
      return null;
    }
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
