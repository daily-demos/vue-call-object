<template>
  <div class="chat-wrapper">
    <button :class="chatIsOpen ? 'chat open' : 'chat'" @click="handleChatClick">
      <template v-if="chatIsOpen">
        <img class="icon" :src="close" alt="" />
      </template>
      <template v-else>
        <img class="icon" :src="chat" alt="" />
      </template>
    </button>
    <div :class="chatIsOpen ? 'chat-container open' : 'chat-container'"></div>
  </div>
</template>

<script>
export default {
  name: "Chat",
  data() {
    return {
      chatIsOpen: false,
      close: require("../assets/x.svg"),
      chat: require("../assets/chat.svg"),
      send: require("../assets/send.svg"),
    };
  },
  methods: {
    handleChatClick() {
      this.chatIsOpen = !this.chatIsOpen;
    },
  },
};
</script>

<style scoped>
.chat-wrapper {
  position: absolute;
  right: 0;
  top: 0;
  height: 100%;
}
.chat-container {
  background-color: #fff;
  width: 300px;
  height: 100%;
  position: absolute;
  right: -300px;
  top: 0;
  transition: right 0.5s ease-out;
}
.chat-container.open {
  right: -0;
}
button.chat {
  background-color: #fff;
  position: absolute;
  right: 0;
  top: calc((100vh - 45px) / 2);
  border: none;
  cursor: pointer;
  transition: right 0.5s ease-out;
}
button.chat.open {
  right: 300px;
}
</style>
