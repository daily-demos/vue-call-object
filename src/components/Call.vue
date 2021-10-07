<template>
  <main>
    <div class="wrapper">
      <h1>call</h1>
      {{ count }}
      <div class="participants-container" v-if="participants">
        <template v-for="p in participants" :key="p.session_id">
          <tile
            :participant="p"
            :handleVideoClick="handleVideoClick"
            :handleAudioClick="handleAudioClick"
          />
        </template>
        <template v-if="count === 1">
          <waiting-card />
        </template>
      </div>
    </div>
  </main>
</template>

<script>
import daily from "@daily-co/daily-js";
import Tile from "./Tile.vue";
import WaitingCard from "./WaitingCard.vue";

const CALL_OPTIONS = {
  url: "https://jessmitch.daily.co/hey",
};

export default {
  name: "Call",
  components: {
    Tile,
    WaitingCard,
  },
  data() {
    return {
      callObject: null,
      participants: null,
      count: 0,
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
    updateParticpants(e) {
      console.log("[EVENT] ", e);
      if (!this.callObject) return;
      const p = this.callObject.participants();
      this.count = Object.values(p).length;
      console.log(this.count);
      this.participants = Object.values(p);
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

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Ropa+Sans&display=swap");
main {
  font-family: "Ropa Sans", sans-serif;
  background-color: #121a24;
  height: 100%;
}
.wrapper {
  max-width: 1200px;
  margin: auto;
  padding: 0 16px;
}
.participants-container {
  display: flex;
  margin: 0 -20px;
  justify-content: center;
}
</style>
