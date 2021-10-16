<template>
  <div>
    <Home />
    <WebGl />
  </div>
</template>

<script>
import Home from "../components/home/index.vue";
import WebGl from "../components/webgl/index.vue";
export default {
  components: {
    Home,
    WebGl,
  },
};
</script>
