<template>
  <nobr>
    <div class="category">
      <span class="categoryName" v-if="!category.editingACategory">{{ category.name }}</span>

      <span v-if="!$parent.showingFriendsPlaces">
        <el-input
          v-if="category.editingACategory"
          v-model="category.name"
          clearable
          :placeholder="`${tmpCategoryName}`"
        >
          <!--<template slot="prepend">Category Name</template>-->
        </el-input>
        <el-button
          v-if="!category.editingACategory"
          type="primary"
          icon="el-icon-edit"
          size="mini"
          circle
          @click="editACategory(category)"
        ></el-button>
        <el-button
          v-if="category.editingACategory"
          type="success"
          icon="el-icon-check"
          circle
          size="mini"
          :disabled="category.name == ''"
          @click="saveACategory(category)"
        ></el-button>
        <el-button
          v-if="category.editingACategory"
          type="warning"
          icon="el-icon-close"
          circle
          size="mini"
          @click="cancelCategoryEdit(category)"
        ></el-button>
        <el-button
          v-if="category.editingACategory"
          type="danger"
          icon="el-icon-delete"
          circle
          size="mini"
          @click="deleteACategory(category)"
        ></el-button>
      </span>
    </div>
  </nobr>
</template>

<script>
export default {
  props: { category: { type: Object, required: true } },
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

<style scoped>
.category {
  width: 85vw !important;
  margin-left: 5vw !important;
}
.categoryName {
  margin-left: 5vw !important;
  margin-right: 2vw !important;
  size: 10px;
  width: 100vw !important;
}
</style>
