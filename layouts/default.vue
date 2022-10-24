<template>
  <v-app style="background: #e4d1b8">
    <v-app-bar
      dense
      app
      clipped-left
      color="rgb(255, 255, 255, 0.1)"
      style="backdrop-filter: blur(4px)"
    >
      <v-toolbar-title v-text="isScreenSmall ? shortTitle : title" />
    </v-app-bar>
    <v-main>
      <v-fade-transition>
        <nuxt />
      </v-fade-transition>
    </v-main>
    <v-footer
      class="font-weight-light text-center text-caption"
      app
      absolute
      tile
      inset
      color="rgb(0, 0, 0, 0)"
    >
      Made with Nuxt.js, powered by Netlify
      <v-spacer></v-spacer>
      <div>&copy; {{ new Date().getFullYear() }}</div>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  computed: {
    isScreenSmall() {
      return this.$vuetify.breakpoint.xsOnly;
    },
  },
  data() {
    return {
      items: [
        {
          icon: "mdi-home",
          title: "Routes",
          to: "/",
        },
        {
          icon: "mdi-book-open-variant",
          title: "Background",
          to: "/background",
        },
        {
          icon: "mdi-hammer-wrench",
          title: "Projects",
          to: "/projects",
        },
        {
          icon: "mdi-playlist-music",
          title: "New Music",
          to: "/new-music",
        },
      ],
      miniVariant: false,
      drawer: false,
      title: "PRISM Interview Challenge",
      shortTitle: "PRISM Interview Challenge",
    };
  },
};
</script>