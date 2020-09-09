<template>
  <nobr>
    <span v-if="!item.editingAnItem">{{ item.name }}</span>
    <el-input v-if="item.editingAnItem" v-model="item.name" clearable :placeholder="`${tmpItemName}`">
      <template slot="prepend">Item Name</template>
    </el-input>
    <el-button
      v-if="!item.editingAnItem"
      type="primary"
      icon="el-icon-edit"
      circle
      @click="editAnItem(item)"
    ></el-button>
    <el-button
      v-if="item.editingAnItem"
      type="success"
      icon="el-icon-check"
      circle
      :disabled="item.name == ''"
      @click="saveAnItem(item)"
    ></el-button>
    <el-button
      v-if="item.editingAnItem"
      type="warning"
      icon="el-icon-close"
      circle
      @click="cancelItemEdit(item)"
    ></el-button>
    <el-button
      v-if="item.editingAnItem"
      type="danger"
      icon="el-icon-delete"
      circle
      @click="deleteAnItem(category, item)"
    ></el-button>
  </nobr>
</template>

<script>
export default {
  props: { category: { type: Object, required: true }, item: { type: Object, required: true } },
  data() {
    return {
      tmpItemName: ''
    }
  },
  created() {},
  methods: {
    editAnItem(item) {
      this.tmpItemName = item.name
      item.editingAnItem = !item.editingAnItem
    },

    cancelItemEdit(item) {
      if (this.tmpItemName) item.name = this.tmpItemName
      item.editingAnItem = !item.editingAnItem
      this.$parent.saveList()
    },

    saveAnItem(item) {
      this.waitingForUSerData = true
      item.editingAnItem = !item.editingAnItem
      this.$parent.saveList()
    },

    deleteAnItem(category, item) {
      if (confirm(`Are you sure you want to delete the item ${item.name}?`) === true) {
        this.waitingForUSerData = true

        for (let i = 0; i < category.items.length; i++) {
          if (category.items[i].name === item.name) {
            category.items.splice(i, 1)
          }
        }

        this.$parent.saveList()
      } else {
        item.editingAnItem = !item.editingAnItem
      }
    }
  }
}
</script>
