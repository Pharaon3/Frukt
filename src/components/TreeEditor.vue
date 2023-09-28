<!-- A tree editor shared by addTree dialog and editTree dialog. -->
<template>
  <v-card>
    <!-- eslint-disable-next-line -->
    <!-- eslint-disable -->
    <v-card-title>
      <slot name="title" />
    </v-card-title>
    <v-card-text>
      <v-form v-model="tree_.valid" @input="$emit('input', tree_)">
        <v-select v-model="tree_.type" required :rules="[v => !!v || 'You must specify a tree variety']"
          :items="insertableTrees" label="Tree variety" />
        <div>
          <v-img v-if="file" :src="previewSource" height="194" alt="New picture" />
          <TreeImage v-else :image="tree_.image" alt="Current image" />
        </div>
        <div v-if="tree_.file && !file">
          <v-btn small class="px-2 mt-1" color="red lighten-3" @click="deleteImage">
            <v-icon>{{ mdiDeleteOutline }}</v-icon>Remove the image
          </v-btn>
        </div>
        <v-file-input v-model="file" accept="image/*" label="Upload a picture" @change="fileChanged" />
        <v-progress-linear :active="uploading" :value="uploadingProgress" :striped="true" />
        <v-text-field v-model="tree_.username" label="Name" :rules="[
          v => {
            if (v && v.trim()) {
              return true
            }
            return 'Please enter your name before continuing.'
          },
        ]" required></v-text-field>
        <v-text-field v-model="tree_.phone" label="Phone Number" :rules="[
          v => !!v || 'Please enter your phone number before continuing',
          v => /^[0-9]{10}$/.test(v) || 'Invalid phone number. Please check again.',
        ]" required></v-text-field>
        <v-text-field v-model="tree_.email" label="Email" :rules="[
          v => !!v || 'Please enter your email address before continuing.',
          v => /.+@.+\..+/.test(v) || 'Invalid E-mail address. Please check again.',
        ]" required></v-text-field>
        <p v-if="file">
          The rotation of the image may be wrong here, but should be correct in the next step.
        </p>
        <v-textarea v-model="tree_.desc" :rules="[
          v => {
            if (v && v.trim()) {
              return true
            }
            return 'Describe the tree before continuing'
          },
        ]" required label="Description" />
      </v-form>
    </v-card-text>
    <v-card-actions>
      <slot name="buttons" />
    </v-card-actions>
    <ConfirmDialog ref="confirm" />
  </v-card>
</template>

<script>
/**
 * A tree editor, for use in dialogs.
 * Implements v-model
 *
 */
import TreeImage from "./TreeImage.vue"
import { mdiDeleteOutline } from "@mdi/js"
import ConfirmDialog from "./ConfirmDialog.vue"

export default {
  name: "TreeEditor",
  components: {
    TreeImage,
    ConfirmDialog,
  },
  model: {
    prop: "tree",
  },
  props: {
    tree: {
      type: Object,
      default: () => ({
        valid: false,
        file: null,
      }),
    },
  },
  data() {
    return {
      tree_: this.tree,
      predefinedTrees: require("../assets/insertableTrees.json"),
      file: null,
      uploading: false,
      uploadingProgress: 0,
      uploadOk: null,

      mdiDeleteOutline,
    }
  },
  computed: {
    insertableTrees() {
      let treeList = Array.from(this.predefinedTrees)
      if (this.tree_.type && !(this.tree_.type in treeList)) {
        treeList.push(this.tree_.type)
      }
      return treeList
    },
    previewSource() {
      return URL.createObjectURL(this.file)
    },
  },
  watch: {
    tree: function (val1, val2) {
      this.tree_ = val2
    },
  },
  methods: {
    deleteImage() {
      this.$refs.confirm
        .open("Are you sure you want to delete the image?")
        .then(confirm => {
          if (confirm) {
            this.tree_.file = null
          }
        })
    },
    fileChanged() {
      if (!this.file) {
        // Reset tree.file in case someone deleted a file they just uploaded
        this.tree_.file = null
      }
      if (this.file) {
        const reader = new FileReader()
        reader.readAsDataURL(this.file)
        reader.onload = () => {
          // let base64String = reader.result.split(",")[1]
          this.tree_.image = reader.result
          console.log("base64String: ", reader.result)
        }
      }
      this.uploading = true
      // fetch(`${process.env.VUE_APP_APIBASE}/sign`, {
      //   method: "POST",
      //   body: JSON.stringify({ "file-name": this.file.name }),
      //   headers: { "Content-Type": "application/json" },
      // })
      //   .then(response => response.json())
      //   .then(response => {
      //     // Rename file. File.name is readonly in most environs
      //     // Creating a new file object works, except for IE and some Opera versions
      //     let renamedFile = new File([this.file], response.filename, {
      //       type: this.file.type,
      //     })
      //     let request = new XMLHttpRequest()
      //     request.open("PUT", response.signedRequest)
      //     request.upload.addEventListener("progress", e => {
      //       this.uploadingProgress = (e.loaded / e.total) * 100
      //     })
      //     request.onreadystatechange = () => {
      //       if (request.readyState === 4) {
      //         this.uploading = false
      //         this.uploadingProgress = 0
      //         if (request.status === 200) {
      //           this.uploadOk = true
      //           // pass
      //         } else {
      //           // FIXME error reporting
      //           this.uploadOk = false
      //           console.log("Error uploading:", request.status)
      //         }
      //       }
      //     }
      //     request.send(renamedFile)
      //     this.tree_.file = response.filename
      //   })
      //   .catch(err => {
      //     // FIXME error reporting
      //     console.log("Error getting upload signature", err)
      //     this.uploading = false
      //     this.uploadingProgress = 0
      //     this.uploadOk = false
      //   })
    },
  },
}
</script>
