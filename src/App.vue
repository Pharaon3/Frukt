<template>
  <v-app id="app">
    <SignUp />
    <SignIn :setAccess="setAccess"/>
    <AppPage :isAdmin="isAdmin" />
  </v-app>
</template>

<script>
import AppPage from "./Page.vue"
import SignUp from "./SignUp.vue"
import SignIn from "./SignIn.vue"

export default {
  name: "App",
  components: {
    AppPage,
    SignUp,
    SignIn,
  },
  data() {
    return {
      isAdmin: Number,
    }
  },
  methods: {
    setAccess(access) {
      this.isAdmin = access
    },
  },
}
</script>

<style>

</style>
