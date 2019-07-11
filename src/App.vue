<template>
  <div id="app">
    <ModulesExample />
  </div>
</template>

<script>
import ModulesExample from "@/components/ModulesExample";

export default {
  components: {
    ModulesExample
  }
};
</script>

<style lang="scss">
#app {
  height: 100vh;
  width: 100vw;
}

// Styles from codepen which are necessary
html,
body {
  margin: 0;
  height: 100%;
  width: 100%;
}
</style>
