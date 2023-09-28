<template>
  <v-app id="signin">
    <v-card>
      <!-- eslint-disable-next-line -->
      <!-- eslint-disable -->
      <h3 class="signin-title">Sign In</h3>
      <v-card-text>
        <v-form @input="$emit('input', sign)">
          <v-text-field v-model="sign.username" label="User Name or Email Address" :rules="[
            v => {
              if (v && v.trim()) {
                return true
              }
              return 'Please enter your name or email address.'
            },
          ]" required></v-text-field>
          <v-text-field v-model="sign.password" label="Password" :rules="[
            v => !!v || 'Please enter your password.',
          ]" type="password" required></v-text-field>
          <v-card-actions class="pt-0">
            <v-btn small @click="handleSignIn"> Sign In </v-btn>
            <v-spacer />
            <v-btn small @click="cancelSignIn">
              Cancel
            </v-btn>
          </v-card-actions>
        </v-form>
      </v-card-text>
    </v-card>
  </v-app>
</template>

<script>
export default {
  name: "SignIn",
  components: {
  },
  props: {
    setAdmin: Function,
  },
  data() {
    return {
      sign: { username: "", password: "" },
    }
  },
  computed: {
  },
  watch: {
  },
  mounted() {
  },
  created: function () {
  },
  beforeDestroy() {
  },
  methods: {
    handleSignIn() {
      let signPayload = this.sign
      signPayload.username = signPayload.username.trim()
      fetch(`${process.env.VUE_APP_APIBASE}/signin`, {
        method: "POST",
        body: JSON.stringify(signPayload),
        headers: {
          "Content-Type": "application/json",
        },
      })
        .then((res) => {
          // Check if the response status is OK (200)
          if (res.ok) {
            return res.json()
          } else {
            throw new Error(`Server returned status ${res.status}`)
          }
        })
        .then((data) => {
          // Handle the JSON data returned by the server
          console.log("Response data:", data)
          if(data.result == "true") {
            document.getElementById("signin").style.display = "none"
            this.setAdmin(data.result.access)
          } else {
            document.getElementById("signin").style.display = "none"
            document.getElementById("signup").style.display = "flex"
          }
          // You can do something with the data here
        })
        .catch(err => {
          // Display error msg, but do not close.
          // User might want to try again
          const msg = "An error occurred while trying to sign in user: " + err
          this.$emit("error", msg)
        })
    },
    cancelSignIn() {
      this.sign.username = ""
      this.sign.password = ""
    },
    
  },
}
</script>

<style>
  #signin {
    display: flex;
    justify-content: center;
    align-items: center;
    background: transparent;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 9;
    backdrop-filter: blur(3px);
  }
  #signin> div {
    max-width: 600px;
    height: 500px !important;
    margin-top: 500px;
    background: transparent;
  }
  .signin-title {
    text-align: center;
    padding: 20px;
    padding-bottom: 0px;
  }
</style>
