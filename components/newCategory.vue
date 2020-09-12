<!-- eslint-disable vue/no-v-html -->
<template>
  <nobr>
    <el-button
      v-if="!addingANewCategory"
      type="success"
      icon="el-icon-plus"
      size="mini"
      round
      @click="startAddANewCategory()"
      >New Category</el-button
    >
    <el-input
      v-if="addingANewCategory"
      v-model="newCategory.name"
      clearable
      size="mini"
      placeholder="Please input a new Category Name"
    >
      <!--<template slot="prepend">New Category Name</template>-->
    </el-input>
    <el-button
      v-if="addingANewCategory"
      type="success"
      icon="el-icon-check"
      circle
      size="mini"
      :disabled="newCategory.name == ''"
      @click="addNewCategory()"
    >
    </el-button>
    <el-button
      v-if="addingANewCategory"
      type="danger"
      icon="el-icon-close"
      circle
      size="mini"
      @click="addingANewCategory = !addingANewCategory"
    >
    </el-button>
  </nobr>
</template>

<script>
export default {
  props: {},
  data() {
    return {
      addingANewCategory: false,
      newCategory: {
        name: '',
        items: [],
        newItem: { name: '', editingAnItem: false },
        addingANewItem: false,
        editingACategory: false
      }
    }
  },
  created() {},
  methods: {
    startAddANewCategory() {
      this.newCategory.name = ''
      this.addingANewCategory = !this.addingANewCategory
    },

    addNewCategory() {
      this.waitingForUSerData = true

      if (this.$parent.list == null) {
        this.$parent.list = [this.newCategory]
      } else {
        this.$parent.list.push(this.newCategory)
      }

      this.addingANewCategory = false

      // Need this bulshit because Javascript is still javascript and it is allways copuing objects by reference and the other cloning methods didn' work, so i need to reset the new category
      this.newCategory = JSON.parse(JSON.stringify(this.newCategory))

      this.$parent.saveList()
    }
  }
}
</script>
