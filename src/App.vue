<script>
import Dashboard from '@/components/Dashboard.vue';

export default {
  name: 'App',
  components: {
    Dashboard
  },
  data() {
    return {
      dataRef: {}
    }
  },
}
</script>

<template>
  <div id="app">
    <Dashboard />
  </div>
</template>

<style lang="scss">
$colorDark: #212225;

body {
  margin: 0px;
  padding: 0px;  
}

#app {
  font-family: Nunito, Roboto, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: white;
  margin: 0px;
  background: $colorDark;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;
z
    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
