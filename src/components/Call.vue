<template>
  <div>
    <h1>call</h1>
    <div v-if="participants">
      <template v-for="p in participants" :key="p.session_id">
        {{ p }}
        <tile
          :participant="p"
          :handleVideoClick="handleVideoClick"
          :handleAudioClick="handleAudioClick"
        />
      </template>
    </div>
  </div>
</template>

<script>
import daily from "@daily-co/daily-js";
import Tile from "./Tile.vue";

const CALL_OPTIONS = {
  url: "https://jessmitch.daily.co/hey",
};

export default {
  name: "Call",
  components: {
    Tile,
  },
  data() {
    return {
      callObject: null,
      participants: null,
    };
  },
  mounted() {
    console.log("here");
    const co = daily.createCallObject(CALL_OPTIONS);
    this.callObject = co;

    co.join();
    co.on("joined-meeting", this.updateParticpants);
    co.on("participant-joined", this.updateParticpants);
    co.on("participant-updated", this.updateParticpants);
    co.on("participant-left", this.updateParticpants);
    co.on("track-started", this.updateParticpants);
    co.on("track-stopped", this.updateParticpants);
    console.log(co);
  },
  methods: {
    updateParticpants() {
      if (!this.callObject) return;
      this.participants = Object.values(this.callObject.participants());
    },
    handleAudioClick() {
      const audioOn = this.callObject.localAudio();
      this.callObject.setLocalAudio(!audioOn);
    },
    handleVideoClick() {
      const videoOn = this.callObject.localVideo();
      this.callObject.setLocalVideo(!videoOn);
    },
  },
};
</script>

<style scoped></style>
