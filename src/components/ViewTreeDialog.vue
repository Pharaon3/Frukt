<template>
  <!-- eslint-disable-next-line -->
  <!-- eslint-disable -->
  <v-dialog
    v-model="displayDialog"
    max-width="440"
    persistent
    @keydown.esc="close"
  >
    <TreeViewer
      v-if="step === 'view'"
      :loading="!Object.entries(tree).length"
      :tree="tree"
    >
      <template #buttons>
        <v-row dense>
          <v-col>
            <v-btn @click="close"> Close </v-btn>
          </v-col>
          <v-spacer />
          <v-col>
            <v-btn :disabled="$store.state.offline" @click="step = 'delete'">
              Delete
            </v-btn>
          </v-col>
          <v-col>
            <v-btn
              :disabled="$store.state.offline"
              @click="
                step = 'edit'
                newTree = { ...tree }
              "
            >
              Edit
            </v-btn>
          </v-col>
        </v-row>
      </template>
    </TreeViewer>

    <TreeEditor v-if="step === 'edit'" v-model="newTree">
      <template #title> Edit tree </template>
      <template #buttons>
        <v-row dense>
          <v-col>
            <v-btn @click="step = 'view'"> Back </v-btn>
          </v-col>
          <v-spacer />
          <v-col>
            <v-btn
              color="green"
              :disabled="!newTree.valid || treesAreEqual(newTree, tree)"
              @click="towardsPreview"
            >
              continue
            </v-btn>
          </v-col>
        </v-row>
      </template>
    </TreeEditor>

    <TreeViewer v-if="step === 'preview'" :tree="newTree" :preview="true">
      <template #buttons>
        <v-row dense>
          <v-col>
            <v-btn @click="step = 'edit'"> Back </v-btn>
          </v-col>
          <v-col>
            <v-btn
              color="green"
              :disabled="$store.state.offline"
              @click="submitTree"
            >
            Save Changes
            </v-btn>
          </v-col>
          <v-spacer />
          <v-col>
            <v-btn @click="close"> Cancel </v-btn>
          </v-col>
        </v-row>
      </template>
    </TreeViewer>

    <v-card v-if="step === 'delete'" :loading="working">
      <v-card-title>Report trees for deletion</v-card-title>
      <v-card-text>
        <p>
          Tell us why this tree should be deleted and one of us will take care of it as soon as possible.
        </p>
        <v-textarea
          v-model="deleteReason"
          :disabled="working"
          :rules="[
            v => {
              if (v && v.trim()) {
                return true
              }
              return 'You need to say why the tree should be deleted'
            },
          ]"
          required
          label="Motivation"
        />
      </v-card-text>
      <v-card-actions>
        <v-row dense>
          <v-col>
            <v-btn @click="step = 'view'"> Back </v-btn>
          </v-col>
          <v-spacer />
          <v-col>
            <v-btn
              color="red"
              :disabled="
                working ||
                $store.state.offline ||
                !deleteReason ||
                !deleteReason.trim()
              "
              @click="deleteOrFlag"
            >
              Report for deletion
            </v-btn>
          </v-col>
        </v-row>
      </v-card-actions>
    </v-card>
    <ConfirmDialog ref="confirm" />
  </v-dialog>
</template>

<script>
import TreeViewer from "./TreeViewer.vue"
import TreeEditor from "./TreeEditor.vue"
import ConfirmDialog from "./ConfirmDialog.vue"
//import { mdiConsoleNetwork } from "@mdi/js"

function raiseOnHttpError(response) {
  if (response.status >= 200 && response.status < 300) {
    return response
  } else {
    let error = new Error(response.statusText)
    error.response = response
    throw error
  }
}

export default {
  name: "ViewTreeDialog",
  components: {
    TreeViewer,
    TreeEditor,
    ConfirmDialog,
  },
  props: {
    value: {
      type: String,
      default: null,
    },
    isAdmin: Number,
    // isAdmin: {
    //   type: Number,
    //   default: 0,
    // },
  },
  data() {
    return {
      step: "view",
      treeCache: {},
      tree: {},
      newTree: {},
      deleteReason: null,
      working: false,
    }
  },
  computed: {
    displayDialog: function () {
      return this.value ? true : false
    },
  },
  /* The dialog is opened before data has been loaded, so we need to watch for
     tree data change, to update the newTree object (used when editing)
  */
  watch: {
    value: function (key) {
      if (!key) {
        this.tree = {}
        return
      }
      this.fetchTree()
    },
  },
  created: function () {
    // If routed here, we'll have a tree from start
    if (this.value) {
      this.fetchTree()
    }
    // if ("tree" in this.$route.params) {
    //    console.log(this.$route.params.tree)
    //    this.viewTree = this.$route.params.tree
    //  }
  },
  methods: {
    close() {
      this.step = "view"
      this.$emit("input", null)
    },
    deleteOrFlag() {
      if(this.isAdmin) this.deleteTree()
      else this.flagForDeletion()
    },
    flagForDeletion() {
      this.working = true
      const key = this.value
      fetch(`${process.env.VUE_APP_APIBASE}/flag/${key}/delete`, {
        method: "POST",
      })
        .then(raiseOnHttpError)
        .then(() => {
          const msg =
            "The tree is marked for deletion. We will look into it as soon as we can."
          this.$emit("info", msg)
        })
        .catch(err => {
          let msg
          if (err.response && err.response.status === 404) {
            msg =
              "You tried to delete a tree that does not exist. " +
              "Maybe it has already been removed?"
          } else {
            msg =
              "Something went wrong while trying to mark this tree for deletion. " +
              err
          }
          this.$emit("error", msg)
        })
        .finally(() => {
          this.working = false
          this.close()
        })
    },
    deleteTree() {
      this.working = true
      const key = this.value
      fetch(`${process.env.VUE_APP_APIBASE}/tree/${key}`, {
        method: "DELETE",
      })
        .then(raiseOnHttpError)
        .then(() => {
          const msg =
            "The tree is marked for deletion. We will look into it as soon as we can."
          this.$emit("info", msg)
        })
        .catch(err => {
          let msg
          if (err.response && err.response.status === 404) {
            msg =
              "You tried to delete a tree that does not exist. " +
              "Maybe it has already been removed?"
          } else {
            msg =
              "Something went wrong while trying to mark this tree for deletion. " +
              err
          }
          this.$emit("error", msg)
        })
        .finally(() => {
          this.working = false
          this.close()
        })
    },
    fetchTree() {
      const key = this.value
      const getData = new Promise(resolve => {
        if (this.treeCache[key]) {
          resolve(this.treeCache[key])
        } else {
          fetch(`${process.env.VUE_APP_APIBASE}/tree/${key}`)
            .then(raiseOnHttpError)
            .then(response => response.json())
            .then(resolve)
            .catch(err => {
              let msg
              if (err.response && err.response.status === 404) {
                msg =
                  "You have followed a link to a tree that does not exist. " +
                  "Maybe it has been removed?"
              } else {
                msg =
                  "Something went wrong when we tried to retrieve this tree. " +
                  err
              }
              this.$emit("error", msg)
              this.close()
            })
        }
      })

      getData
        .then(data => {
          this.$emit("tree-loaded", data)
          this.tree = data
          this.treeCache[key] = this.tree
        })
        .catch(err => {
          const msg = "We were unable to retrieve this tree at this time. " + err
          this.$emit("error", msg)
          this.close()
        })
    },

    towardsPreview() {
      // tree is edited, we require confirmation if changing tree type
      if (this.tree.type === this.newTree.type) {
        this.step = "preview"
        return
      }
      this.$refs.confirm
        .open(
          `Do you want to change the tree's type from 
            ${this.tree.type} to ${this.newTree.type}?`,
          { positiveText: "continue", positiveColor: "green" }
        )
        .then(confirm => {
          if (confirm) {
            this.step = "preview"
          }
        })
    },

    submitTree() {
      let key = this.value
      let treePayload = this.newTree
      treePayload.type = treePayload.type.trim()
      treePayload.desc = treePayload.desc.trim()
      fetch(`${process.env.VUE_APP_APIBASE}/tree/${this.value}`, {
        method: "POST",
        body: JSON.stringify(treePayload),
        headers: {
          "Content-Type": "application/json",
        },
      })
        .then(() => {
          delete this.treeCache[key]
          this.$emit("change") // trigger map refresh, in case tree type changed
          this.close()
        })
        .catch(err => {
          // Display error msg, but do not close.
          // User might want to try again
          const msg = "An error occurred while trying to update the tree: " + err
          this.$emit("error", msg)
        })
    },

    /* Check if there are differences we care about betw trees*/
    treesAreEqual(tree1, tree2) {
      const type1 = tree1.type ? tree1.type.trim() : ""
      const type2 = tree2.type ? tree2.type.trim() : ""
      const desc1 = tree1.desc ? tree1.desc.trim() : ""
      const desc2 = tree2.desc ? tree2.desc.trim() : ""
      return type1 === type2 && desc1 === desc2 && tree1.file === tree2.file
    },
  },
}
</script>
