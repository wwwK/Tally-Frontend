<template>
  <v-app-bar app collapse-on-scroll :value="isShow">
    <v-icon class="mx-4" x-large color="#007AE9"> mdi-notebook-edit</v-icon>
    <v-app-bar-title class="subtitle-1" v-text="title"></v-app-bar-title>
  </v-app-bar>
</template>
<script>
export default {
  name: "AppBar",
  props: {
    isShow: Boolean
  },
  data: function () {
    return {
      title: this.GLOBAL.title
    };
  },
  mounted() {
  },
};
</script>
