<template>
  <app-header />
  <home v-if="appState === 'idle'" :joinCall="joinCall" />
  <call v-else-if="appState === 'incall'" />
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
    };
  },
  methods: {
    joinCall() {
      console.log("click");
      this.appState = "incall";
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
