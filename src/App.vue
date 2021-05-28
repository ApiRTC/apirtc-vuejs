<template>
  <p>
    <img alt="ApiRTC logo" src="./assets/ApiRTC.png" width="66" height="76" /> &
    <img alt="Vue logo" src="./assets/logo.png" width="76" height="76" />
  </p>
  <Conversation />
</template>

<script>
import Conversation from "./components/Conversation.vue";

export default {
  name: "App",
  components: {
    Conversation,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
