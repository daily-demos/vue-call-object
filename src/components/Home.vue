<template>
  <main>
    <div class="wrapper">
      <h1>Call object Vue demo</h1>
      <p>Demo a custom call interface built using Daily call object for Vue</p>

      <form class="join-form" @submit="joinWithName">
        <div class="name-container">
          <label for="name">Name</label>
          <input type="text" id="name" required v-model="name" />
        </div>
        <div class="submit-container">
          <button type="submit">
            Start demo
          </button>
        </div>
      </form>

      <p class="subtext">
        Select "Allow" to use your camera and mic for this call
      </p>
    </div>
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
  data() {
    return {
      name: "",
    };
  },
  methods: {
    joinWithName() {
      this.joinCall(this.name);
    },
  },
};
</script>

<style scoped>
main {
  display: flex;
  justify-content: center;
  align-items: center;
  flex: 1;
  background-color: #fff;
}
.wrapper {
  max-width: 1200px;
  margin: auto;
  padding: 0 16px;
}
.join-form {
  display: flex;
  flex-direction: column;
  width: 300px;
  margin: auto;
}

.join-form label {
  font-size: 12px;
  color: #6b7785;
  text-align: left;
  margin-bottom: 4px;
}
.join-form input {
  padding: 8px;
  border: 1px solid #c8d1dc;
  border-radius: 12px;
}
.join-form div {
  display: flex;
  justify-content: center;
}
.join-form .submit-container button {
  padding: 7px 16px 9px;
  font-size: 12px;
  font-weight: bold;
  border: none;
  border-radius: 12px;
  margin-top: 16px;
  margin-bottom: 12px;
  background-color: #1bebb9;
  cursor: pointer;
}
.join-form .name-container {
  flex-direction: column;
}
.subtext {
  font-size: 12px;
  color: #6b7785;
}
</style>
