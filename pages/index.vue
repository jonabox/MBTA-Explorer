<template>
  <v-layout>
    <v-container fluid>
      <v-tabs
        v-model="tab"
        class="mb-4"
        color="black"
        background-color="transparent"
        grow
      >
        <v-tab>Exercise 1</v-tab>
        <v-tab>Exercise 2</v-tab>
        <v-tab>Exercise 3</v-tab>
      </v-tabs>
      <v-tabs-items style="background-color: transparent" v-model="tab">
        <!-- Exercise 1 -->
        <v-tab-item>
          <v-row>
            <v-col
              v-for="(routes, index) in [lightRailRoutes, heavyRailRoutes]"
              :key="index"
            >
              <h1
                v-text="
                  index == 0 ? 'Light Rail Routes:' : 'Heavy Rail Routes:'
                "
              ></h1>
              <v-card
                v-for="route in routes"
                :key="route.attributes.long_name"
                color="rgb(255, 255, 255, 0.1)"
                class="mx-1 my-4"
              >
                <v-sheet
                  :color="'#' + route.attributes.color"
                  height="5"
                  width="100%"
                ></v-sheet>
                <v-card-title
                  style="word-break: normal"
                  v-text="route.attributes.long_name"
                />
                <v-card-subtitle
                  v-text="
                    route.attributes.direction_destinations[0] +
                    ' ⇆ ' +
                    route.attributes.direction_destinations[1]
                  "
                />
              </v-card>
            </v-col>
          </v-row>
        </v-tab-item>
        <!-- Exercise 2 -->
        <v-tab-item>
          <v-row>
            <v-col cols="auto">
              <h1>Stop Counts</h1>
              <v-card
                v-for="(stopsInRoute, index) in stopCounts"
                :key="stopsInRoute.route"
                color="rgb(255, 255, 255, 0.1)"
                class="mx-1 my-4"
              >
                <v-sheet
                  :color="'#' + stopsInRoute.color"
                  height="5"
                  width="100%"
                ></v-sheet>
                <v-card-title
                  style="word-break: normal"
                  v-text="stopsInRoute.name"
                />
                <v-card-subtitle>
                  {{
                    stopsInRoute.count +
                    " stops" +
                    minAndMaxSuffix(index, stopCounts)
                  }}
                </v-card-subtitle>
              </v-card>
            </v-col>
            <v-col cols="10">
              <h1>Connections</h1>
              <v-container>
                <v-row>
                  <template
                    v-for="[stop, connectedRoutes] in stopToConnectedRoutes"
                  >
                    <v-card
                      v-if="connectedRoutes.length > 1"
                      :key="stop"
                      color="rgb(255, 255, 255, 0.1)"
                      class="mx-1 my-4"
                      width="500"
                    >
                      <v-card-title style="word-break: normal" v-text="stop" />
                      <v-chip
                        v-for="route in connectedRoutes"
                        :key="route.id"
                        class="ma-1"
                        outlined
                      >
                        {{ route.attributes.long_name }}
                        <v-icon right :color="'#' + route.attributes.color">
                          mdi-train
                        </v-icon>
                      </v-chip>
                    </v-card>
                  </template>
                </v-row>
              </v-container>
            </v-col>
          </v-row>
        </v-tab-item>
        <!-- Exercise 3 -->
        <v-tab-item>
          <h1>Route Finder (Solution might not be optimal!)</h1>
          <v-row>
            <v-col>
              <v-autocomplete
                label="start"
                auto-select-first
                :items="stops"
                v-model="start"
              />
            </v-col>
            <v-col>
              <v-autocomplete
                label="destination"
                auto-select-first
                :items="stops"
                v-model="destination"
              />
            </v-col>
            <v-col>
              <v-btn @click="findRoute()"> Find Route </v-btn>
            </v-col>
          </v-row>
          <v-row v-if="routingResult">
            <v-timeline class="ma-4">
              <v-timeline-item
                v-for="route in routingResult"
                color="white"
                :key="route"
                >{{ route }}</v-timeline-item
              >
            </v-timeline>
          </v-row></v-tab-item
        >
      </v-tabs-items>
    </v-container>
  </v-layout>
</template>

<script>
export default {
  data() {
    return {
      lightRailRoutes: [],
      heavyRailRoutes: [],
      stops: [],
      stopCounts: [],
      stopToConnectedRoutes: new Map(),
      routesToStops: new Map(),
      routesToAdjacentRoutes: new Map(),
      start: null,
      destination: null,
      routingResult: null,
      tab: null,
    };
  },
  mounted() {
    this.getRoutes();
  },
  methods: {
    // fetches light and heavy routes in the MBTA
    getRoutes() {
      let data = {
        headers: {
          accept: "application/vnd.api+json",
        },
        params: {
          "filter[type]": "0,1",
        },
      };
      this.$axios
        .get("https://api-v3.mbta.com/routes", data)
        .then((response) => {
          for (let route of response.data["data"]) {
            // process stops for each route
            this.getStops(route);
            // type-0 and type-1 correspond to light and heavy rails, respectively
            if (route.attributes.type == 0) {
              this.lightRailRoutes.push(route);
            } else if (route.attributes.type == 1) {
              this.heavyRailRoutes.push(route);
            }
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // fetches all the stops for a specific route in the MBTA
    getStops(route) {
      let data = {
        headers: {
          accept: "application/vnd.api+json",
        },
        params: {
          "filter[route]": route.id,
          include: "connecting_stops",
        },
      };
      this.$axios
        .get("https://api-v3.mbta.com/stops", data)
        .then((response) => {
          let routeName = route.attributes.long_name;
          let routeColor = route.attributes.color;
          let routeStopCount = response.data["data"].length;
          this.generateStopCountForRoute(routeName, routeColor, routeStopCount);
          this.findConnectingRoutes(route, response);
          this.generateRouteMap();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // stores the number of stops for the route
    generateStopCountForRoute(routeName, routeColor, routeStopCount) {
      this.stopCounts.push({
        name: routeName,
        color: routeColor,
        count: routeStopCount,
      });
      // sort routes by number of stops; decreasing order
      this.stopCounts.sort(function (a, b) {
        return b.count - a.count;
      });
    },
    // find stops that connect to one or more
    findConnectingRoutes(route, response) {
      const routeName = route.attributes.long_name;
      for (let stop of response.data["data"]) {
        let stopName = stop.attributes.name;
        this.stops.push(stopName);
        // add stop to route map
        if (this.routesToStops.has(routeName)) {
          let stops = this.routesToStops.get(routeName);
          stops.push(stopName);
        } else {
          let stops = [stopName];
          this.routesToStops.set(routeName, stops);
        }
        // check if same stop has already been found
        if (this.stopToConnectedRoutes.has(stopName)) {
          let connectedRoutes = this.stopToConnectedRoutes.get(stopName);
          connectedRoutes.push(route);
        } else {
          // else create connected routes array for current stop
          let connectedRoutes = [route];
          this.stopToConnectedRoutes.set(stopName, connectedRoutes);
        }
      }
    },
    // generate graph of routes connecting to other routes
    generateRouteMap() {
      for (let [stop, connectedRoutes] of this.stopToConnectedRoutes) {
        for (let route1 of connectedRoutes) {
          let routeName1 = route1.attributes.long_name;
          for (let route2 of connectedRoutes) {
            let routeName2 = route2.attributes.long_name;
            if (routeName1 != routeName2) {
              // check if route already exists in mapping
              if (this.routesToAdjacentRoutes.has(routeName1)) {
                let adjacentRoutes =
                  this.routesToAdjacentRoutes.get(routeName1);
                adjacentRoutes.add(routeName2);
              }
              // else create set of connecting routes
              else {
                let adjacentRoutes = new Set();
                adjacentRoutes.add(routeName2);
                this.routesToAdjacentRoutes.set(routeName1, adjacentRoutes);
              }
            }
          }
        }
      }
    },
    // finds route between start and destination using DFS
    findRoute() {
      // get routes for start and destinations
      // arbitrarily pick the 0th route if multiple exist
      let start = this.stopToConnectedRoutes.get(this.start)[0];
      let startName = start.attributes.long_name;
      let destination = this.stopToConnectedRoutes.get(this.destination)[0];
      let destinationName = destination.attributes.long_name;
      let DFS = (currentRoute, pathSoFar) => {
        // check if we have arrived at destination
        if (currentRoute == destinationName) {
          return pathSoFar;
        }
        // else continue through DFS
        else {
          // get neighbors for current route
          let neighbors = this.routesToAdjacentRoutes.get(currentRoute);
          for (let neighbor of neighbors) {
            // exclude previously visited neighbors
            if (!pathSoFar.includes(neighbor)) {
              let result = DFS(neighbor, [...pathSoFar, neighbor]);
              // check for truthy value i.e. valid solution
              if (result) {
                return result;
              }
            }
          }
        }
      };
      let pathSoFar = [startName];
      this.routingResult = DFS(startName, pathSoFar);
    },

    minAndMaxSuffix(index, stopCounts) {
      if (index == 0) {
        return " (highest)";
      } else if (index == stopCounts.length - 1) {
        return " (lowest)";
      } else {
        return "";
      }
    },
  },
  computed: {
    isScreenSmall() {
      return this.$vuetify.breakpoint.mdAndDown;
    },
  },
};
</script>