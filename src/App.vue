<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
    <h2>Minesweeper</h2>
    </v-app-bar>
    <v-main>
      <MineSweeper/>
    </v-main>
  </v-app>
</template>

<script>
import MineSweeper from './components/MineSweeper';

export default {
  name: 'App',

  components: {
    MineSweeper,
  },

  data: () => ({
    //
  }),
};
</script>
