<template>
  <div class="newItem">
    <img src="/img/items/0.jpg" alt="" width="130" height="70" style="object-fit: cover; width: 130px; height: 70px" />
    <br />
    <nobr>
      <br />
      <el-button
        v-if="!category.addingANewItem"
        type="primary"
        icon="el-icon-plus"
        size="mini"
        round
        @click="category.addingANewItem = !category.addingANewItem"
      >
        New Item in {{ category.name }}
      </el-button>
      <el-input
        v-if="category.addingANewItem"
        v-model="category.newItem.name"
        clearable
        size="mini"
        :placeholder="`Please enter a new item in ${category.name}`"
      >
        <!--<template slot="prepend">New Item Name</template>-->
      </el-input>
      <br />
      <el-button
        v-if="category.addingANewItem"
        type="success"
        icon="el-icon-check"
        circle
        size="mini"
        :disabled="category.newItem.name == ''"
        @click="addNewItem(category)"
      ></el-button>
      <el-button
        v-if="category.addingANewItem"
        type="danger"
        icon="el-icon-close"
        circle
        size="mini"
        @click="category.addingANewItem = !category.addingANewItem"
      ></el-button>
    </nobr>
  </div>
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

      for (let i = 0; i < this.$parent.user.list.length; i++) {
        if (this.$parent.user.list[i].name === category.name) {
          this.$parent.user.list[i].items.push(category.newItem)
        }
      }

      category.newItem = { name: '', editingAnItem: false }

      this.$parent.saveList()
      category.addingANewItem = false
    }
  }
}
</script>
