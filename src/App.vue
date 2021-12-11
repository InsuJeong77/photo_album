<template>
  <v-app>
    <v-main>
      <photo-control/>
    </v-main>
  </v-app>
</template>

<script>
import PhotoControl from './components/PhotoControl';

export default {
  name: 'App',

  components: {
    PhotoControl,
  },

  data: () => ({
    //
  }),
};
</script>
