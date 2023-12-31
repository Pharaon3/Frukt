<template>
  <v-app id="page">
    <v-navigation-drawer
      v-model="drawer"
      :mobile-breakpoint="960"
      :height="100"
      :mini-variant="miniVariant"
      right
      bottom
      absolute
    >
      <v-card flat>
        <h1>
          <v-card-title class="hidden-lg-and-up">
            <v-img
              :src="mLogo[1]"
              :srcset="`${mLogo[1]} 1x,${mLogo[2]} 2x,${mLogo[3]} 3x`"
              :max-width="280"
            />
          </v-card-title>
          <v-card-title class="hidden-md-and-down" style="padding: 0">
            <v-img
              v-if="!miniVariant"
              alt="The fruit map"
              src="./assets/img/fruktkartan_a.png"
              style="height: 100px; width: 140px;"
            >
              <span class="beta" :style="{ display: betaDisplay }">beta</span>
            </v-img>
            <v-img v-else alt="The fruit map" src="./assets/img/f_t.png" />
          </v-card-title>
        </h1>
      </v-card>

      <v-list style="display: flex; flex-direction: row;">
        <SidebarItem
          :icon-img="selectedTreeIcon"
          :tooltip="`Shows ${selectedTreeName.toLowerCase()}`"
          @mini-action="miniVariant = false"
        >
          <v-select
            v-model="filters.type"
            :items="selectTreeTypes"
            label="Select tree to display"
          />
        </SidebarItem>
        <SidebarItem
          :icon="mdiPlus"
          :active="
            $refs.map && ($refs.map.addTree || $refs.map.addTreeMarker.visible)
          "
          :disabled="
            $store.state.offline ||
            ($refs.map && $refs.map.addTreeMarker.visible)
          "
          @on-click="$refs.map.addNewTree()"
        >
          Add trees
        </SidebarItem>
        <SidebarItem :icon="mdiImagePlus" :active="showAddMapBulk" to="/addmapbulk">
          Add Fruit Map in Bulk
        </SidebarItem>
        <SidebarItem :icon="mdiInformation" :active="showFAQ" to="/om">
          About the Fruit Map
        </SidebarItem>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <TheMap
        ref="map"
        class="fill-height"
        :tree-filters="filters"
        :isAdmin="isAdmin" 
        @open-drawer="drawer = true"
        @close-drawer="drawer = null"
      />
      <AboutUs v-model="showFAQ" />
      <AddMapBulk v-model="showAddMapBulk" />
      <v-snackbar v-model="offlineWarning" :timeout="-1" color="warning">
        We could not find any internet connection at this time. 
        You will not be able to add or edit trees until you are connected.
        <template #action="{ attrs }">
          <v-btn text v-bind="attrs" @click="offlineWarning = false">
            Close
          </v-btn>
        </template>
      </v-snackbar>
    </v-main>
  </v-app>
</template>

<script>
import TheMap from "./components/Map.vue"
import AboutUs from "./components/About.vue"
import AddMapBulk from "./components/AddMapBulk.vue"
import SidebarItem from "./components/SidebarItem.vue"
import {
  mdiClose,
  mdiMenuOpen,
  mdiFilter,
  mdiReload,
  mdiInformation,
  mdiImagePlus,
  mdiPlus,
} from "@mdi/js"

const DEFAULT_FILTERS = {
  type: "*",
}

const mobileLogotype = {
  // Vue does not support relative imports in srcset
  // See https://github.com/vuejs-templates/webpack/issues/396
  1: require("./assets/img/fruktkartan_m.png"),
  2: require("./assets/img/fruktkartan_m2.png"),
  3: require("./assets/img/fruktkartan_m3.png"),
}

export default {
  name: "AppPage",
  components: {
    TheMap,
    AboutUs,
    AddMapBulk,
    SidebarItem,
  },
  props: {
    isAdmin: Number,
  },
  data() {
    return {
      offlineWarning: this.$store.state.offline,

      showFAQ: false,
      showAddMapBulk: false,

      /* v-navigation-drawer */
      drawer: null, // null means closed on mobile, open on desktop
      miniVariant: false,
      betaDisplay: "none",

      /* tree filters */
      selectTreeTypes: require("./assets/selectTrees.json"),
      filters: { ...DEFAULT_FILTERS },

      /* SVG icons */
      mdiClose,
      mdiMenuOpen,
      mdiFilter,
      mdiReload,
      mdiInformation,
      mdiImagePlus,
      mdiPlus,
    }
  },
  computed: {
    console: () => console, // For debugging
    selectedTreeIcon() {
      let tree = this.filters.type === "*" ? "tree" : this.filters.type
      return require(`./components/ico/${tree}.svg`)
    },
    selectedTreeName() {
      let treeKey = this.filters.type
      if (treeKey === "*") {
        return "all trees"
      } else {
        return this.selectTreeTypes.filter(x => x.value === treeKey)[0].text
      }
    },
    mLogo() {
      return mobileLogotype
    },
  },
  watch: {
    $route: function (r) {
      // Check for special routes
      this.showFAQ = r.path === "/om"
      this.showAddMapBulk = r.path === "/addmapbulk"
      if ("goatcounter" in window) {
        // send a page view to goatcounter
        // window.goatcounter.allow_local = true
        window.goatcounter.count({
          path: () => r.path,
          event: true,
        })
      }
    },
    showFAQ: function (state) {
      // HACK
      // Checking if the dialog was closed, but the route didn't change
      // This is most likely not the way to do it...
      if (!state && this.$route.path === "/om") {
        this.$router.push("/")
      }
    },
    showAddMapBulk: function (state) {
      // HACK
      // Checking if the dialog was closed, but the route didn't change
      // This is most likely not the way to do it...
      if (!state && this.$route.path === "/addmapbulk") {
        this.$router.push("/")
      }
    },
  },
  mounted() {
    // There might be a vue plugin for this, but on the other hand quite straightforward
    window.addEventListener("online", this.updateOnlineStatus)
    window.addEventListener("offline", this.updateOnlineStatus)
  },
  created: function () {
    if (window.location.hostname != "fruktkartan.se") {
      this.betaDisplay = "block"
    }
    if (this.$route.path === "/om") {
      this.showFAQ = true
    }
    if (this.$route.path === "/addmapbulk") {
      this.showAddMapBulk = true
    }

    /* Hack around unintended effects of using 100vh in some mobile browser, see
    https://stackoverflow.com/questions/37112218/css3-100vh-not-constant-in-mobile-browser
    */
    function appHeight() {
      const doc = document.documentElement
      doc.style.setProperty("--vh", window.innerHeight * 0.01 + "px")
    }
    window.addEventListener("resize", appHeight)
    appHeight()
  },
  beforeDestroy() {
    window.removeEventListener("online", this.updateOnlineStatus)
    window.removeEventListener("offline", this.updateOnlineStatus)
  },
  methods: {
    updateOnlineStatus({ type }) {
      if (type === "offline") {
        this.$store.commit("offline", true)
        this.offlineWarning = true
      } else {
        this.$store.commit("offline", false)
        this.offlineWarning = false
      }
    },
    reset() {
      this.filters = { ...DEFAULT_FILTERS }
    },
  },
}
</script>

<style>
@import "../node_modules/leaflet/dist/leaflet.css";

/* Hack around unintended effects of using 100vh in some mobile browser, see
https://stackoverflow.com/questions/37112218/css3-100vh-not-constant-in-mobile-browser
*/
.v-application {
  height: 100vh;
  height: -webkit-fill-available;
  height: calc(var(--vh, 1vh) * 100);
  min-height: calc(var(--vh, 1vh) * 100);
  max-height: calc(var(--vh, 1vh) * 100);
}
.v-application--wrap {
  height: inherit !important;
  max-height: inherit !important;
  min-height: inherit !important;
}

html,
body {
  height: 100vh;
  height: -webkit-fill-available;
}

html {
  overflow: hidden !important;
}

.beta {
  display: none;
  float: right;
  padding: 0.25em;
  color: black;
  background-color: #6db6e1;
  border-radius: 50%;
}

/* Override uppercasing of text buttons (but not the flat) */
.v-btn--contained {
  text-transform: none !important;
}

/* Less whitespace on mobile */
.v-navigation-drawer--is-mobile {
  height: auto !important;
}
aside {
  width: 100vw !important;
  display: flex !important;
  flex-direction: row !important;
}
aside .v-navigation-drawer__content {
  display: flex;
  flex-direction: row;
  /* width: 80%; */
  overflow: hidden;
}
aside .v-navigation-drawer__content .v-card {
  /* width: 30%; */
}
aside> .v-navigation-drawer__append> .d-lg-block {
  display: flex !important;
  flex-direction: row !important;
}
aside .v-card .v-image__image {
  width: 100px;
}
.v-list-item__icon {
  height: 100%;
  margin: 0;
}
#page {
  display: none;
}
</style>
