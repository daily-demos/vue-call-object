<template>
  <div class="card">
    <h2>Waiting for others!</h2>
    <h3>Invite someone by sharing this link:</h3>
    <p>make this a link ;)</p>
  </div>
</template>

<script>
export default {
  name: "WaitingCard",
};
</script>

<style>
.card {
  background-color: #2b3f56;
  border-radius: 16px;
  max-width: 480px;
  margin: 0 20px;
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.card h2 {
  color: #1bebb9;
  margin: 2px 0;
}
.card h3 {
  color: #fff;
  margin: 2px 0;
}
.card p {
  color: #fff;
  margin: 16px 0 0;
  font-size: 12px;
}
</style>
