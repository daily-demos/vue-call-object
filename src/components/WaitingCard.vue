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
  min-width: 480px;
}
.card h2 {
  color: #1bebb9;
}
.card h3,
.card p {
  color: #fff;
}
</style>
