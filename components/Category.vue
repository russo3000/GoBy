<template>
  <nobr>
    <span v-if="!category.editingACategory">{{ category.name }}</span>
    <el-input v-if="category.editingACategory" v-model="category.name" clearable :placeholder="`${tmpCategoryName}`">
      <template slot="prepend">Category Name</template>
    </el-input>
    <el-button
      v-if="!category.editingACategory"
      type="primary"
      icon="el-icon-edit"
      size="medium"
      circle
      @click="editACategory(category)"
    ></el-button>
    <el-button
      v-if="category.editingACategory"
      type="success"
      icon="el-icon-check"
      circle
      @click="saveACategory(category)"
      :disabled="category.name == ''"
    ></el-button>
    <el-button
      v-if="category.editingACategory"
      type="warning"
      icon="el-icon-close"
      circle
      @click="cancelCategoryEdit(category)"
    ></el-button>
    <el-button
      v-if="category.editingACategory"
      type="danger"
      icon="el-icon-delete"
      circle
      @click="deleteACategory(category)"
    ></el-button>
  </nobr>
</template>

<script>
export default {
  props: { category: { name: '', items: [], editingACategory: false } },
  data() {
    return {
      tmpCategoryName: ''
    }
  },
  created() {},
  methods: {
    deleteACategory(category) {
      if (confirm(`Are you sure you want to delete the category ${category.name}?`) === true) {
        this.waitingForUSerData = true

        for (let i = 0; i < this.$parent.list.length; i++) {
          if (this.$parent.list[i].name === category.name) {
            this.$parent.list.splice(i, 1)
          }
        }

        this.$parent.saveList()
      } else {
        category.editingACategory = !category.editingACategory
      }
    },

    saveACategory(category) {
      category.editingACategory = !category.editingACategory
      this.$parent.saveList()
    },
    editACategory(category) {
      this.tmpCategoryName = category.name
      category.editingACategory = !category.editingACategory
    },
    cancelCategoryEdit(category) {
      if (this.tmpCategoryName) category.name = this.tmpCategoryName
      category.editingACategory = !category.editingACategory
      this.$parent.saveList()
    }
  }
}
</script>
