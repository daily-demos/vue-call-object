<template>
  <div class="controls">
    <button @click="handleAudioClick">
      <template v-if="participant.audio">
        <img class="icon" :src="micOn" alt="" />
      </template>
      <template v-else>
        <img class="icon" :src="micOn" alt="" />
      </template>
    </button>
    <button @click="handleVideoClick">
      <template v-if="participant.video">
        <img class="icon" :src="videoOn" alt="" />
      </template>
      <template v-else>
        <img class="icon" :src="videoOn" alt="" />
      </template>
    </button>
    <button @click="leaveCall">
      <img class="icon" :src="leave" alt="" />
    </button>
  </div>
</template>

<script>
export default {
  name: "Controls",
  props: ["participant", "handleVideoClick", "handleAudioClick", "leaveCall"],
  data() {
    return {
      leave: require("../assets/leave_call.svg"),
      micOn: require("../assets/mic_on.svg"),
      screenshare: require("../assets/screenshare.svg"),
      videoOn: require("../assets/vid_on.svg"),
    };
  },
};
</script>

<style scoped>
.controls {
  position: absolute;
  bottom: 16px;
  left: 8px;
  border-radius: 12px;
  background-color: #121a24;
  padding: 8px;
}
button {
  background-color: transparent;
  border: none;
  cursor: pointer;
}
button:hover {
}
.icon {
  height: 24px;
}
</style>
