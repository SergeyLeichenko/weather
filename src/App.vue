<template>
  <div id="app">
    <Header />

    <div class="tabs">
      <Tabs />
    </div>

  </div>
</template>

<script>
import Header from './components/HeaderComponent.vue'
import Tabs from './components/TabsComponent.vue'

export default {
  name: 'App',
  components: {
    Header,
    Tabs
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
.tabs {
  background: rgba(45,176,224,1);
  background: -moz-linear-gradient(45deg, rgba(45,176,224,1) 0%, rgba(96,40,168,1) 100%);
  background: -webkit-gradient(left bottom, right top, color-stop(0%, rgba(45,176,224,1)), color-stop(100%, rgba(96,40,168,1)));
  background: -webkit-linear-gradient(45deg, rgba(45,176,224,1) 0%, rgba(96,40,168,1) 100%);
  background: -o-linear-gradient(45deg, rgba(45,176,224,1) 0%, rgba(96,40,168,1) 100%);
  background: -ms-linear-gradient(45deg, rgba(45,176,224,1) 0%, rgba(96,40,168,1) 100%);
  background: linear-gradient(45deg, rgba(45,176,224,1) 0%, rgba(96,40,168,1) 100%);
  filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#2db0e0', endColorstr='#6028a8', GradientType=1 );
  padding: 20px;
}
</style>
