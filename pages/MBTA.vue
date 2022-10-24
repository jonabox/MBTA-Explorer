<template>
  <v-layout>
    <v-flex>
      <v-card flat color="transparent" class="ma-3"> Hello </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
export default {
  components: {
    Logo,
  },
  data() {
    return {
      lightRailRoutes: [],
      heavyRailRoutes: [],
    };
  },
  mounted() {
    this.getRoutes();
  },
  methods: {
    getRoutes() {
      let data = {
        headers: {
          accept: "application/vnd.api+json",
        },
        params: {
          "filter[type]":"0,1"
        },
      };
      axios
        .get("https://api-v3.mbta.com/routes", data)
        .then((response) => {
          console.log(response.data);
          // let items = response.data.albums.items;
          // items.sort(sortByDate);
          // // empty albums
          // this.albums = [];
          // this.uniqueAlbums = new Set();
          // for (let album of response.data.albums.items) {
          //   if (!this.uniqueAlbums.has(album.name)) {
          //     this.uniqueAlbums.add(album.name);
          //     this.albums.push(album);
          //   }
          // }
          // this.nextURL = response.data.albums.next;
          // this.areAlbumsLoaded = true;
          
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
  computed: {
    isScreenSmall() {
      return this.$vuetify.breakpoint.mdAndDown;
    },
  },
};
</script>