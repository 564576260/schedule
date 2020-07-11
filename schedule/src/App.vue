<template>
  <div id="app">
    <schedule/>
  </div>
</template>

<script>
import schedule from './components/schedule.vue'

export default {
  name: 'App',
  components: {
    schedule
  }
}
</script>

<style>
*{
	margin: 0;
	padding: 0;
	list-style: none;
}
.clearfix:after {
  content: " ";
  display: block;
  clear: both;
  height: 0;
}
.clearfix {
  zoom: 1;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;margin: 0;
	padding: 0;
	list-style: none;
	
}
</style>
