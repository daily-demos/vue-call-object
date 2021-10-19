<template>
  <app-header />
  <home v-if="appState === 'idle'" :joinCall="joinCall" />
  <call
    v-else-if="appState === 'incall'"
    :leaveCall="leaveCall"
    :name="name"
    :roomUrl="roomUrl"
  />
</template>

<script>
import Call from "./Call.vue";
import AppHeader from "./AppHeader.vue";
import Home from "./Home.vue";

export default {
  name: "App",
  components: {
    Call,
    AppHeader,
    Home,
  },
  data() {
    return {
      appState: "idle",
      name: "Guest",
      roomUrl: null,
    };
  },
  methods: {
    joinCall(name, url) {
      this.name = name;
      this.roomUrl = url;
      this.appState = "incall";
    },
    leaveCall() {
      this.appState = "idle";
    },
  },
};
</script>

<style>
#app {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow-x: hidden;
  background-color: #121a24;
}
a {
  text-decoration: none;
  color: #2c3e50;
  display: flex;
  align-items: center;
}
body {
  margin: 0;
}
</style>
