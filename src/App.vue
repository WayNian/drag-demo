<template>
  <div id="app">
    <DragView></DragView>
  </div>
</template>
<script>
import DragView from "./components/DragView.vue";
export default {
  components: {
    DragView,
  },
};
</script>
<style lang="scss">
* {
  margin: 0;
  box-sizing: border-box;
}
</style>
