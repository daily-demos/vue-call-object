<template>
  <header>
    <div>
      <div class="row">
        <a class="right-border" href="https://daily.co">
          <img class="icon" :src="daily" alt="Daily.co" />
        </a>
        <img class="icon" src="../assets/logo.png" alt="Daily.co" />
        <p>Call object Vue Demo</p>
      </div>
      <div class="row">
        <div class="right-border">
          <a class="button-link" href="" rel="noreferrer noopener">
            <span>API docs</span>
            <img :src="newtab" alt="Open in new tab" />
          </a>
        </div>
        <img class="icon" :src="github" alt="Github repo" />
      </div>
    </div>
  </header>
</template>

<script>
export default {
  name: "AppHeader",
  data() {
    return {
      github: require("../assets/github.svg"),
      daily: require("../assets/daily.svg"),
      newtab: require("../assets/newtab.svg"),
    };
  },
};
</script>

<style>
header {
  font-size: 12px;
  line-height: 16px;
  font-weight: bold;
  border-bottom: 1px solid #c8d1dc;
  padding: 2px;
}
header > div {
  max-width: 1200px;
  margin: auto;
  padding: 0 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.icon {
  height: 24px;
}
.row {
  display: flex;
  align-items: center;
}
.right-border {
  padding-right: 16px;
  border-right: 1px solid #c8d1dc;
  margin-right: 16px;
}
.button-link {
  border: 1px solid #c8d1dc;
  border-radius: 8px;
  padding: 7px 16px;
}
@media screen and (max-width: 700px) {
  .button-link {
    display: none;
  }
}
</style>
