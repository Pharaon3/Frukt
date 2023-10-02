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
      <v-file-input
        v-model="selectedFile"
        accept=".csv"
        @change="handleFileUpload"
        label="Upload CSV File"
        class="file-input"
      ></v-file-input>
      <v-card-actions>
        <v-btn @click.stop="addTree"> OK </v-btn>
        <v-btn @click.stop="close"> Close </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import Papa from "papaparse"

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
      selectedFile: null,
      importedTrees: [],
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
    handleFileUpload() {
      if (this.selectedFile) {
        // Read the selected file (CSV) here and process it as needed
        // You can use JavaScript libraries like PapaParse to parse CSV data
        // For example:
        Papa.parse(this.selectedFile, {
          header: true,
          complete: (results) => {
            console.log(results.data)
            this.importedTrees = results.data
          },
        })
      }
    },
    addTree() {
      console.log("this.importedTrees: ", this.importedTrees)
      this.importedTrees.map((tree) => {
        let treePayload = {
          username: tree.EIER,
          lat: tree.Lat,
          lon: tree.Long,
          desc: tree.treplassering,
          type: tree.TYPE,
          valid: true
        }
        treePayload.type = treePayload.type.trim()
        treePayload.desc = treePayload.desc.trim()
        console.log("treePayload: ", treePayload)
        console.log("process.env.VUE_APP_APIBASE: ", process.env.VUE_APP_APIBASE)
        fetch(`${process.env.VUE_APP_APIBASE}/tree`, {
          method: "PUT",
          body: JSON.stringify(treePayload),
          headers: { "Content-Type": "application/json" },
        })
          .then(() => {
            this.step = "edit"
            this.tree = {}
            this.$emit("input", false)
            this.$emit("added")
          })
          .catch(err => {
            const msg =
              "We were unable to save the tree at this time: " +
              err +
              "\nPlease try again in a while!"
            this.$emit("error", msg)
          })
        console.log("treePayLoad: ", treePayload)
      })
    },
  },
}
</script>
<style>
  .file-input {
    padding: 0 24px 20px;
  }
</style>