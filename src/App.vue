<template>
  <div id="app">
    <div class="wrapper">
      <InputForm />
    </div>
  </div>
</template>

<script>
import InputForm from "./components/InputForm.vue";

export default {
  name: "App",
  components: {
    InputForm
  }
};
</script>

<style>
#app {
  font-family: Noto Sans;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  /* background: #010415; */
}

.wrapper {
  width: 100vw;
  height: 100vh;
  background: #010415;
}
</style>
