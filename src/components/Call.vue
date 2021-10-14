<template>
  <main>
    <div class="wrapper">
      <p>{{ count }}</p>

      <template v-if="error">
        <p class="error-text">{{ error }}</p>
      </template>

      <div class="participants-container" v-if="participants">
        <template v-for="p in participants" :key="p.session_id">
          <video-tile
            :participant="p"
            :handleVideoClick="handleVideoClick"
            :handleAudioClick="handleAudioClick"
            :leaveCall="leaveAndCleanUp"
          />
        </template>

        <template v-if="count === 1">
          <waiting-card :url="url" />
        </template>
      </div>

      <chat :sendMessage="sendMessage" :messages="messages" />
    </div>
  </main>
</template>

<script>
import daily from "@daily-co/daily-js";

import WaitingCard from "./WaitingCard.vue";
import Chat from "./Chat.vue";
import VideoTile from "./VideoTile.vue";

export default {
  name: "Call",
  components: {
    VideoTile,
    WaitingCard,
    Chat,
  },
  props: ["leaveCall", "name"],
  data() {
    return {
      callObject: null,
      participants: null,
      count: 0,
      messages: [],
      error: false,
      loading: false,
      url: "https://jessmitch.daily.co/hey",
    };
  },
  mounted() {
    const option = {
      url: this.url,
    };
    const co = daily.createCallObject(option);
    this.callObject = co;

    co.join({ userName: this.name });

    // Add call and participant event handler
    co.on("joined-meeting", this.updateParticpants)
      .on("joining-meeting", this.handleJoiningMeeting)
      .on("participant-joined", this.updateParticpants)
      .on("participant-updated", this.updateParticpants)
      .on("participant-left", this.updateParticpants)
      .on("track-started", this.updateParticpants)
      .on("track-stopped", this.updateParticpants)
      .on("error", this.handleError)
      .on("app-message", this.updateMessages);
  },
  methods: {
    updateParticpants(e) {
      console.log("[EVENT] ", e);
      if (!this.callObject) return;
      const p = this.callObject.participants();
      this.count = Object.values(p).length;
      this.participants = Object.values(p);
      this.loading = false;
    },
    updateMessages(e) {
      console.log("[MESSAGE] ", e.data);
      this.messages.push(e?.data);
    },
    handleError(e) {
      console.log("[ERROR] ", e);
      this.error = e?.errorMsg;
      this.loading = false;
    },
    handleJoiningMeeting(e) {
      console.log("[JOINING] ", e);
      this.loading = true;
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
      // Messages are sent local and received by everyone, so set the username when it's sent.
      const local = this.callObject.participants().local;
      const message = { message: text, name: local?.user_name || "Guest" };
      this.messages.push(message);
      this.callObject.sendAppMessage(message, "*");
    },
    leaveAndCleanUp() {
      this.callObject.leave().then(() => {
        this.callObject.destroy();

        this.leaveCall();
      });
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
  flex-wrap: wrap;
}
p {
  color: white;
}
.error-text {
  color: #e71115;
}
</style>
