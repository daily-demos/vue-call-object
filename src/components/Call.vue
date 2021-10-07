<template>
  <main>
    <div class="wrapper">
      <p>{{ count }}</p>
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
      <chat />
    </div>
  </main>
</template>

<script>
import daily from "@daily-co/daily-js";
import Tile from "./Tile.vue";
import WaitingCard from "./WaitingCard.vue";
import Chat from "./Chat.vue";

const CALL_OPTIONS = {
  url: "https://jessmitch.daily.co/hey",
};

export default {
  name: "Call",
  components: {
    Tile,
    WaitingCard,
    Chat,
  },
  data() {
    return {
      callObject: null,
      participants: null,
      count: 0,
      messages: [],
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
    co.on("app-message", this.updateMessages);
    console.log(co);
  },
  methods: {
    updateParticpants(e) {
      console.log("[EVENT] ", e);
      if (!this.callObject) return;
      const p = this.callObject.participants();
      this.count = Object.values(p).length;
      this.participants = Object.values(p);
    },
    updateMessages(e) {
      console.log("[MESSAGE] ", e);
    },
    handleAudioClick() {
      const audioOn = this.callObject.localAudio();
      this.callObject.setLocalAudio(!audioOn);
    },
    handleVideoClick() {
      const videoOn = this.callObject.localVideo();
      this.callObject.setLocalVideo(!videoOn);
    },
    sendMessage(text) {
      this.messages.push({ text });
      this.callObject.sendAppMessage({ message: text }, "*");
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
  position: relative;
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
p {
  color: white;
}
</style>
