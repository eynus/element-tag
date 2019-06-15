<template>
  <div id="app">
    <div class="container">
      <my-tag></my-tag>
      
    </div>
  </div>
</template>

<script>
import myTag from "./components/Tag";
export default {
  name: "app",
  components: {
    myTag,
  }
};
</script>

<style scoped>
#app {
  width: 100%;
  min-height: 100vh;
  background-color: #999;
  display: flex;
  justify-content: center;
  align-items: center;
}
.container {
  width: 500px;
  height: 400px;
  padding: 10px 15px;
  background-color: #fff;
}
</style>
