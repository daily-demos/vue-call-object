<template>
  <Header />
  <Call />
</template>

<script>
import Call from "./Call.vue";
import Header from "./Header.vue";

export default {
  name: "App",
  components: {
    Call,
    Header,
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
