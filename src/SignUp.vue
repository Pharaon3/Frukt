<template>
  <v-app id="signup">
    <v-card>
      <!-- eslint-disable-next-line -->
      <!-- eslint-disable -->
      <h3 class="signup-title">Sign Up</h3>
      <v-card-text>
        <v-form @input="$emit('input', sign)">
          <v-text-field v-model="sign.username" label="User Name" :rules="[
            v => {
              if (v && v.trim()) {
                return true
              }
              return 'Please enter your name.'
            },
          ]" required></v-text-field>
          <v-text-field v-model="sign.email" label="Email" :rules="[
            v => !!v || 'Please enter your email address.',
            v => /.+@.+\..+/.test(v) || 'Invalid E-mail address. Please check again.',
          ]" required></v-text-field>
          <v-text-field v-model="sign.password" label="Password" :rules="[
            v => !!v || 'Please enter your password.',
          ]" type="password" required></v-text-field>
          <v-card-actions class="pt-0">
            <v-btn small @click="handleSignUp"> Sign Up </v-btn>
            <v-spacer />
            <v-btn small @click="gotoSignIn"> Go to Sign In </v-btn>
            <v-spacer />
            <v-btn small @click="cancelSignUp">
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
  name: "SignUp",
  components: {
  },
  data() {
    return {
      sign: { username: "", email: "", password: "" },
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
    handleSignUp() {
      let signPayload = this.sign
      signPayload.username = signPayload.username.trim()
      signPayload.email = signPayload.email.trim()
      fetch(`${process.env.VUE_APP_APIBASE}/signup`, {
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
            document.getElementById("signup").style.display = "none"
            document.getElementById("page").style.display = "block"
          } else {
            this.sign.username = ""
            this.sign.email = ""
            this.sign.password = ""
          }
          // You can do something with the data here
        })
        .catch(err => {
          // Display error msg, but do not close.
          // User might want to try again
          const msg = "An error occurred while trying to sign up new user: " + err
          this.$emit("error", msg)
        })
    },
    cancelSignUp() {
      this.sign.username = ""
      this.sign.email = ""
      this.sign.password = ""
    },
    gotoSignIn() {
      this.sign.username = ""
      this.sign.email = ""
      this.sign.password = ""
      document.getElementById("signin").style.display = "flex"
      document.getElementById("signup").style.display = "none"
    },
    
  },
}
</script>

<style>
  #signup {
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
    display: none;
  }
  #signup> div {
    max-width: 600px;
    height: 500px !important;
    margin-top: 500px;
    background: transparent;
  }
  .signup-title {
    text-align: center;
    padding: 20px;
    padding-bottom: 0px;
  }
  
</style>
