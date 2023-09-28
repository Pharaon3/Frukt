<template>
  <!-- eslint-disable-next-line -->
  <!-- eslint-disable -->
  <v-dialog v-model="displayDialog" max-width="500" @keydown.esc="close">
    <v-card>
      <v-card-title>Add Fruit Map in Bulk</v-card-title>
      <v-card-text>
        <p>
          You can import Fruit Map in CSV format.
        </p>
      </v-card-text>
      <v-card-actions>
        <v-btn @click.stop="close"> Close </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: "AddMapBulk",
  props: {
    value: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      timestamp: process.env.VUE_APP_TIMESTAMP ?? "?",
      githash: process.env.VUE_APP_GITHASH ?? "?",
    }
  },
  computed: {
    gitcommit: function () {
      return "https://github.com/fruktkartan/fruktkartan/commit/" + this.githash
    },
    displayDialog: {
      get: function () {
        return this.value ? true : false
      },
      set: function (val) {
        this.$emit("input", val)
      },
    },
  },
  methods: {
    close() {
      this.$emit("input", false)
    },
  },
}
</script>
