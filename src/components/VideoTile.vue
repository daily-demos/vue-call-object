<template>
  <div class="tile">
    <audio autoPlay playsInline :srcObject="audioSource">
      <track kind="captions" />
    </audio>

    <template v-if="videoTrack">
      <video autoPlay muted playsInline :srcObject="videoSource"></video>
      <p class="participant-name">{{ username }}</p>
    </template>

    <template v-else>
      <no-video-tile :username="username"></no-video-tile>
    </template>

    <template v-if="local">
      <call-controls
        :audio="audioTrack"
        :disable-screen-share="disableScreenShare"
        :handle-audio-click="handleAudioClick"
        :handle-screenshare-click="handleScreenshareClick"
        :handle-video-click="handleVideoClick"
        :leave-call="leaveCall"
        :video="videoTrack"
      />
    </template>
  </div>
</template>

<script lang="ts">
import CallControls from "./CallControls.vue";
import NoVideoTile from "./NoVideoTile.vue";

export default {
  name: "VideoTile",
  components: {
    CallControls,
    NoVideoTile,
  },
  props: [
    "audioTrack",
    "local",
    "screen",
    "sessionId",
    "userName",
    "videoTrack",
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
    console.log("mounmted participant", this.participant);
    this.handleVideo(this.videoTrack);
    // this.handleAudio(this.audioTrack);
  },
  updated() {
    // this.username = this.participant?.user_name;
    // // For later optimization, this can be done more selectively
    // // using "track-started" and "track-stopped" events.
    // this.handleVideo(this.participant.videoTrack);
    // this.handleAudio(this.participant);
  },
  methods: {
    // Add srcObject to video element
    handleVideo(videoTrack) {
      console.log("handleVideo", videoTrack);

      // If the participant has their video off,
      // early out.
      if (!videoTrack) return;

      const newStream = this.updateSource(this.videoSource, videoTrack);
      if (newStream) {
        this.videoSource = newStream;
      }
    },

    // Add srcObject to audio element
    handleAudio() {
      const p = this.participant;
      // If the participant is local or has their audio off,
      // early out.
      if (!p || p.local || !p.audio) return;

      const track = p.tracks.audio.persistentTrack;
      const newStream = this.updateSource(this.audioSource, track);
      if (newStream) {
        this.audioSource = newStream;
      }
    },

    // updateSource() updates the given stream with new tracks OR
    // returns an entirely new stream to the caller.
    updateSource(stream, newTrack) {
      const existingTracks = stream?.getTracks();
      // If the stream parameter contains no existing tracks,
      // just return a new MediaStream to set. This should
      // only happen the first time the tile is initialized.
      if (!existingTracks || existingTracks.length === 0) {
        return new MediaStream([newTrack]);
      }
      if (existingTracks.length > 1) {
        console.warn(
          `expected 1 track, found ${existingTracks.length}. Only using the first one.`
        );
      }
      const existingTrack = existingTracks[0];
      // If existing track is different from the new track,
      // remove the existing track and add the new one.
      if (newTrack.id !== existingTrack.id) {
        stream.removeTrack(existingTrack);
        stream.addTrack(newTrack);
      }
      return null;
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
