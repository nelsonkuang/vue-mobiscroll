<template>
  <div id="app">
    <div class="wrap">
      <scroller :scrollerTitle="vTitle" :itemList="vList"></scroller>
    </div>
  </div>
</template>

<script>
import Scroller from './components/Scroller';

export default {
  name: 'app',
  data() {
    return {
      vList: [
        { value: 1, name: 'item 1', selected: false },
        { value: 2, name: 'item 2', selected: false },
        { value: 3, name: 'item 3', selected: false },
        { value: 4, name: 'item 4', selected: true },
        { value: 5, name: 'item 5', selected: false },
        { value: 6, name: 'item 6', selected: false },
        { value: 7, name: 'item 7', selected: false },
        { value: 8, name: 'item 8', selected: false },
      ],
      vTitle: 'List Title',
    };
  },
  components: {
    Scroller,
  },
};
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0,0,0,0.5);
}
.wrap {
  width: 300px;
  background-color: #fff;
  position: relative;
}
</style>
