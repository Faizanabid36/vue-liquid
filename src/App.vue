<template>
  <div id="app">  
    <CsvRender/>
  </div>
</template>

<script>
import CsvRender from "./components/CsvRender";
import Vue from "vue";
import Vuetify from "vuetify";
import colors from "vuetify/es5/util/colors";

import VueClipboard from "vue-clipboard2";
VueClipboard.config.autoSetContainer = true;
Vue.use(VueClipboard);

Vue.use(Vuetify, {
  theme: {
    primary: '#3f51b5',
    secondary: '#b0bec5',
    accent: '#8c9eff',
    error: '#b71c1c'
  }
});

export default {
  name: "App",
  components: {
    CsvRender
  }
};
</script>

<style>
#app {
  
}
</style>
