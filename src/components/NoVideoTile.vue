<template>
  <div class="no-video">
    <p>no video</p>
  </div>
</template>

<script>
export default {
  name: "NoVideoTile",
};
</script>

<style>
.no-video {
  background-color: #2b3f56;
  border-radius: 16px;
  aspect-ratio: 16/9;
  height: 100%;
  color: #fff;
  position: relative;
  display: flex;
  flex: 1;
  justify-content: center;
  align-items: center;
  padding: 2px 0;
}
</style>
