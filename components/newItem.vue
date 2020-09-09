<template>
  <nobr>
    <el-button
      v-if="!category.addingANewItem"
      type="primary"
      icon="el-icon-plus"
      size="medium"
      round
      @click="category.addingANewItem = !category.addingANewItem"
    >
      New Item in {{ category.name }}
    </el-button>
    <el-input
      v-if="category.addingANewItem"
      v-model="category.newItem.name"
      clearable
      :placeholder="`Please enter a new item in ${category.name}`"
    >
      <template slot="prepend">New Item Name</template>
    </el-input>
    <el-button
      v-if="category.addingANewItem"
      type="success"
      icon="el-icon-check"
      circle
      :disabled="category.newItem.name == ''"
      @click="addNewItem(category)"
    ></el-button>
    <el-button
      v-if="category.addingANewItem"
      type="danger"
      icon="el-icon-close"
      circle
      @click="category.addingANewItem = !category.addingANewItem"
    ></el-button>
  </nobr>
</template>

<script>
export default {
  props: { category: { type: Object, required: true } },
  data() {
    return {}
  },
  created() {},
  methods: {
    addNewItem(category) {
      this.waitingForUSerData = true

      for (let i = 0; i < this.$parent.list.length; i++) {
        if (this.$parent.list[i].name === category.name) {
          if (this.$parent.list[i].items.length > 0) {
            this.$parent.list[i].items.unshift(category.newItem)
          } else {
            this.$parent.list[i].items.push(category.newItem)
          }
        }
      }

      category.newItem = { name: '', editingAnItem: false }

      this.$parent.saveList()
      category.addingANewItem = false
    }
  }
}
</script>
