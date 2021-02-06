<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      Torrent Theatre

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <Theatre/>
    </v-main>
  </v-app>
</template>

<script>
import Theatre from './components/Theatre';

export default {
  name: 'App',

  components: {
    Theatre,
  },

  data: () => ({
    //
  }),
};
</script>
