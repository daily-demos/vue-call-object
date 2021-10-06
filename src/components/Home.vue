<template>
  <main>
    <h1>Call object Vue demo</h1>
    <p>Demo a custom call interface built using Daily call object for Vue</p>
    <button @click="joinCall">
      Start demo
    </button>
    <p>Select "Allow" to use your camera and mic for this call</p>
  </main>
</template>

<script>
export default {
  name: "Home",
  props: {
    joinCall: {
      type: Function,
    },
  },
};
</script>

<style></style>
