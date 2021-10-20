<template>
  <div class="controls">
    <div class="devices">
      <button @click="handleAudioClick">
        <template v-if="participant.audio">
          <img class="icon" :src="micOn" alt="" />
        </template>
        <template v-else>
          <img class="icon" :src="micOff" alt="" />
        </template>
      </button>

      <button @click="handleVideoClick">
        <template v-if="participant.video">
          <img class="icon" :src="videoOn" alt="" />
        </template>
        <template v-else>
          <img class="icon" :src="videoOff" alt="" />
        </template>
      </button>

      <template v-if="supportsScreenshare">
        <button @click="handleScreenshareClick" :disabled="disableScreenShare">
          <img class="icon" :src="screenshare" alt="" />
        </button>
      </template>
    </div>

    <button class="leave" @click="leaveCall">
      <img class="icon" :src="leave" alt="" />
    </button>
  </div>
</template>

<script>
import daily from "@daily-co/daily-js";

export default {
  name: "Controls",
  props: [
    "participant",
    "handleVideoClick",
    "handleAudioClick",
    "handleScreenshareClick",
    "leaveCall",
    "disableScreenShare"
  ],
  data() {
    return {
      leave: require("../assets/leave_call.svg"),
      micOn: require("../assets/mic_on.svg"),
      micOff: require("../assets/mic_off.svg"),
      screenshare: require("../assets/screenshare.svg"),
      videoOn: require("../assets/vid_on.svg"),
      videoOff: require("../assets/vid_off.svg"),
      supportsScreenshare: false,
    };
  },
  mounted() {
    this.supportsScreenshare = daily.supportedBrowser().supportsScreenShare;
  },
};
</script>

<style scoped>
.controls {
  position: absolute;
  bottom: 12px;
  left: 8px;
  justify-content: space-between;
  display: flex;
  width: calc(100% - 16px);
}

.devices {
  border-radius: 12px;
  background-color: #121a24;
  opacity: 0.85;
  padding: 14px 10px 15px;
}
button {
  background-color: transparent;
  border: none;
  cursor: pointer;
}
button:not(.leave) {
  margin: 0 4px;
}
button.leave {
  background-color: #f63135;
  opacity: 0.85;
  padding: 14px 16px 15px;
  border-radius: 12px;
}
button:disabled {
  cursor: not-allowed;
  opacity: 0.4;
}
.icon {
  height: 24px;
}
</style>
