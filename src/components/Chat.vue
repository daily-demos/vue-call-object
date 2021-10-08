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
    <div :class="chatIsOpen ? 'chat-container open' : 'chat-container'">
      <div class="messages">
        <p v-for="(chat, i) in messages" :key="i">{{ chat?.message }}</p>
      </div>
      <form @submit="submitForm">
        <div class="input">
          <label for="message">Type a message...</label>
          <textarea
            id="message"
            type="text"
            @keyup.enter="submitForm"
            v-model="text"
            placeholder="Type a message..."
          />
        </div>
        <button class="submit-button" type="submit">
          <img :src="send" alt="" />
        </button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: "Chat",
  props: ["sendMessage", "messages"],
  data() {
    return {
      chatIsOpen: false,
      close: require("../assets/x.svg"),
      chat: require("../assets/chat.svg"),
      send: require("../assets/send.svg"),
      text: "",
    };
  },
  methods: {
    handleChatClick() {
      this.chatIsOpen = !this.chatIsOpen;
    },
    submitForm(e) {
      e.preventDefault();
      this.sendMessage(e?.target?.value);
      this.text = "";
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Ropa+Sans&display=swap");

.chat-wrapper {
  position: absolute;
  right: 0;
  top: 0;
  height: 100%;
}
.chat-container {
  background-color: #fff;
  width: 300px;
  height: calc(100% - 48px);
  position: absolute;
  right: -348px;
  top: 0;
  transition: right 0.5s ease-out;
  display: flex;
  flex-direction: column;
  padding: 24px;
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
  border-radius: 16px 0 0 16px;
  padding: 16px 14px 13px 18px;
}
button.chat.open {
  right: 348px;
}

.messages {
  flex: 1;
}
.input {
  display: flex;
  flex-direction: column;
  flex: 1;
  align-items: flex-start;
}
label {
  color: #fff;
}
.input textarea {
  width: 100%;
  border: none;
  resize: none;
  font-family: "Ropa Sans", sans-serif;
  font-size: 16px;
}
.input textarea::placeholder {
  font-family: "Ropa Sans", sans-serif;
  font-size: 16px;
}
form {
  display: flex;
  border-bottom: 2px solid #c8d1dc;
}

.submit-button {
  padding: 4px;
  margin: 0 0 0 16px;
  border: none;
  background-color: #fff;
}
</style>
