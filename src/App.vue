<template>
  <div id="app">
    <img src="./assets/logo.png">
    <file-input></file-input>
  </div>
</template>

<script>
import FileInput from './components/FileInput';

export default {
  name:       'app',
  components: {
    FileInput,
  },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
