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


::-webkit-scrollbar {
    width: 24px !important;
    
}

::-webkit-scrollbar-track {
    background: #212225 !important;
    border: none !important;
}

::-webkit-scrollbar-thumb {
    border: 9px solid rgba(0, 0, 0, 0) !important;
    background-clip: padding-box !important;
    -webkit-border-radius: 24px !important;
    background-color: white !important;
    //-webkit-box-shadow: inset -1px -1px 0px rgba(0, 0, 0, 0.05), inset 1px 1px 0px rgba(0, 0, 0, 0.05);

    &:hover {
      opacity: 0.4;
    }
}

.dark-mode::-webkit-scrollbar-thumb {
    background: #2a2b2e !important;
    border-radius: 24px !important;
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
