<template>
  <div class="app">
    <div v-if="user.isLoggedIn">
      <img :src="user.photoURL" class="avatar" />

      <div>Last Login: {{ currentUserLastLogin }} <button @click="logout()">Logout</button></div>

      <br />
      <br />
      <ul>
        <li>
          <nobr>
            <el-button
              v-if="!addingANewCategory"
              type="success"
              icon="el-icon-plus"
              size="medium"
              round
              @click="startAddANewCategory()"
              >New Category</el-button
            >
            <el-input
              v-if="addingANewCategory"
              v-model="newCategory.name"
              clearable
              placeholder="Please input a new Category Name"
            >
              <template slot="prepend">New Category Name</template>
            </el-input>
            <el-button
              v-if="addingANewCategory"
              type="success"
              icon="el-icon-check"
              circle
              @click="addNewCategory()"
              :disabled="newCategory.name == ''"
            >
            </el-button>
            <el-button
              v-if="addingANewCategory"
              type="danger"
              icon="el-icon-close"
              circle
              @click="addingANewCategory = !addingANewCategory"
            >
            </el-button>
          </nobr>
        </li>
        <li v-for="category in list" :key="category.id">
          <nobr>
            <span v-if="!category.editingACategory">{{ category.name }}</span>
            <el-input
              v-if="category.editingACategory"
              v-model="category.name"
              clearable
              :placeholder="`${tmpCategoryName}`"
            >
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
          <ul>
            <li>
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
                  @click="addNewItem(category)"
                  :disabled="category.newItem.name == ''"
                ></el-button>
                <el-button
                  v-if="category.addingANewItem"
                  type="danger"
                  icon="el-icon-close"
                  circle
                  @click="category.addingANewItem = !category.addingANewItem"
                ></el-button>
              </nobr>
            </li>
            <li v-for="item in category.items" :key="item.id">
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
                  @click="saveAnItem(item)"
                  :disabled="item.name == ''"
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
            </li>
          </ul>
        </li>
      </ul>
      <br />
      <br />
      <br />
      <br />
    </div>

    <div v-else-if="waitingForUSerData" class="lds-spinner">
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
    </div>

    <button v-if="!waitingForUSerData && !user.isLoggedIn" @click="login('google')">Sign In with Google</button>
    <button v-if="!waitingForUSerData && !user.isLoggedIn" @click="login('facebook')">Sign In with FB</button>
  </div>
</template>

<!-- The core Firebase JS SDK is always required and must be listed first -->
<script src="/__/firebase/7.19.0/firebase-app.js"></script>
<script src="/__/firebase/7.19.0/firebase-analytics.js"></script>
<!--  <script src="/__/firebase/init.js"></script> -->

<script>
/* eslint-disable no-unused-vars */
/* eslint-disable prefer-const */
/* eslint-disable no-console */
/* eslint-disable no-var */

import firebase from 'firebase/app'
import 'firebase/auth'
import 'firebase/firestore'

// When a user logs in or out, save that in the store

export default {
  components: {},
  data: () => {
    return {
      tmpCategoryName: '',
      tmpItemName: '',
      addingANewCategory: false,
      list: null,
      newCategory: {
        name: '',
        items: [],
        newItem: { name: '', editingAnItem: false },
        addingANewItem: false,
        editingACategory: false
      },

      user: {
        isLoggedIn: false,
        displayName: 'Not Logged In', // Placeholders for what google will return
        photoURL: '' // Placeholders for what google will return
      },
      waitingForUSerData: false,
      currentUserLastLogin: '',
      firebaseConfig: {
        apiKey: 'AIzaSyB_s6xHy0taW3IPki18O01VR9SFVpvfDIk',
        authDomain: 'goby-71f9f.firebaseapp.com',
        databaseURL: 'https://goby-71f9f.firebaseio.com',
        projectId: 'goby-71f9f',
        storageBucket: 'goby-71f9f.appspot.com',
        messagingSenderId: '716714440093',
        appId: '1:716714440093:web:2934bd7abf6939620b2b74',
        measurementId: 'G-1SC55Y36HZ'
      },
      fb: null,
      firebaseDb: null
    }
  },

  created() {
    this.fb = firebase

    // Need this cuz of this strange bug i can't find any info about
    if (!this.fb.apps.length) {
      this.fb.initializeApp(this.firebaseConfig)
    }

    this.firebaseDb = this.fb.firestore()

    this.waitingForUSerData = true

    // Here is the Firebase Authentication Magic happening returning a FireBase authenticated User
    this.fb.auth().onAuthStateChanged((fbuser) => {
      if (fbuser != null) {
        this.user = fbuser
        this.user.isLoggedIn = true

        // Options to use Firebase Get command
        var getOptions = { source: 'default' }

        this.firebaseDb
          .collection('login')
          .doc(this.user.uid)
          .get(getOptions)
          .then((doc) => {
            this.currentUserLastLogin = doc.data().lastLogin
            this.getList()
            this.waitingForUSerData = false
          })
          .catch(function (error) {
            console.log('Error getting cached document:', error)
          })
      } else {
        this.user.isLoggedIn = false
        this.waitingForUSerData = false
      }
    })
  },

  methods: {
    login(providerType) {
      var provider = null

      switch (providerType) {
        case 'facebook':
          provider = new this.fb.auth.FacebookAuthProvider()
          break
        case 'google':
          provider = new this.fb.auth.GoogleAuthProvider()
          break
        default:
          alert(providerType + 'Not Supported')
          break
      }

      this.fb
        .auth()
        .signInWithRedirect(provider)
        .then()
        .catch(function (error) {
          console.log(error)
        })
    },
    logout() {
      let currentdate = new Date().toLocaleString()
      this.firebaseDb
        .collection('login')
        .doc(this.user.uid)
        .set({ lastLogin: this.user.displayName + ' : ' + currentdate })
        .then(() => {
          console.log('Last Logout successfully written to firebase')

          this.fb
            .auth()
            .signOut()
            .then(() => {
              this.user.isLoggedIn = false
              this.waitingForUSerData = false

              location.reload()
            })
            .catch(function (error) {
              console.log(error)
            })

          this.waitingForUSerData = false
        })
    },

    saveList() {
      if (this.list) {
        this.list.map((category) => {
          // Recetting the state of all objects
          this.addingANewCategory = false
          category.editingACategory = false
          category.addingANewItem = false

          if (category.items) {
            category.items.map((item) => {
              item.editingAnItem = false
            })
          }

          this.firebaseDb
            .collection('list')
            .doc(this.user.uid)
            .set({ list: this.list })
            .then(() => {
              this.waitingForUSerData = false
            })
        })
      }
    },

    getList() {
      this.waitingForUSerData = true
      // Optionf to user Firebase Get command
      var getOptions = {
        source: 'default'
      }

      this.firebaseDb
        .collection('list')
        .doc(this.user.uid)
        .get(getOptions)
        .then((doc) => {
          this.list = doc.data().list

          if (this.list) {
            this.list.map((category) => {
              category.addingANewItem = false
              category.editingACategory = false
              category.newItem = { name: '', editingAnItem: false }
            })

            this.waitingForUSerData = false
          } else {
            this.list = null
          }
        })
        .catch(function (error) {
          console.log('Error getting cached document:', error)
        })
    },

    startAddANewCategory() {
      this.newCategory.name = ''
      this.addingANewCategory = !this.addingANewCategory
    },

    addNewCategory() {
      this.waitingForUSerData = true

      if (this.list == null) {
        this.list = [this.newCategory]
      } else {
        this.list.unshift(this.newCategory)
      }

      this.addingANewCategory = false

      //Need this bulshit because Javascript is still javascript and it is allways copuing objects by reference and the other cloning methods didn' work, so i need to reset the new category
      this.newCategory = JSON.parse(JSON.stringify(this.newCategory))

      this.saveList()
    },

    deleteACategory(category) {
      if (confirm(`Are you sure you want to delete the category ${category.name}?`) == true) {
        this.waitingForUSerData = true

        for (var i = 0; i < this.list.length; i++) {
          if (this.list[i].name === category.name) {
            this.list.splice(i, 1)
          }
        }

        this.saveList()
      } else {
        category.editingACategory = !category.editingACategory
      }
    },

    saveACategory(category) {
      category.editingACategory = !category.editingACategory
      this.saveList()
    },
    editACategory(category) {
      this.tmpCategoryName = category.name
      category.editingACategory = !category.editingACategory
    },
    cancelCategoryEdit(category) {
      if (this.tmpCategoryName) category.name = this.tmpCategoryName
      category.editingACategory = !category.editingACategory
      this.saveList()
    },

    addNewItem(category) {
      this.waitingForUSerData = true

      for (var i = 0; i < this.list.length; i++) {
        if (this.list[i].name === category.name) {
          if (this.list[i].items.length > 0) {
            this.list[i].items.unshift(category.newItem)
          } else {
            this.list[i].items.push(category.newItem)
          }
        }
      }

      category.newItem = { name: '', editingAnItem: false }

      this.saveList()
      category.addingANewItem = false
    },

    deleteAnItem(category, item) {
      if (confirm(`Are you sure you want to delete the item ${item.name}?`) == true) {
        this.waitingForUSerData = true

        for (var i = 0; i < category.items.length; i++) {
          if (category.items[i].name === item.name) {
            category.items.splice(i, 1)
          }
        }

        this.saveList()
      } else {
        item.editingAnItem = !item.editingAnItem
      }
    },

    saveAnItem(item) {
      this.waitingForUSerData = true
      item.editingAnItem = !item.editingAnItem
      this.saveList()
    },

    editAnItem(item) {
      this.tmpItemName = item.name
      item.editingAnItem = !item.editingAnItem
    },

    cancelItemEdit(item) {
      if (this.tmpItemName) item.name = this.tmpItemName
      item.editingAnItem = !item.editingAnItem
      this.saveList()
    }
  }
}
</script>
