<template>
  <div>
    <button @click="handleAudioClick">
      {{ participant.audio ? "Mute" : "Unmute" }}
    </button>
    <button @click="handleVideoClick">
      {{ participant.video ? "Mute v" : "Unmute v" }}
    </button>
  </div>
</template>

<script>
export default {
  name: "Controls",
  props: ["participant", "handleVideoClick", "handleAudioClick"],
};
</script>

<style scoped></style>
