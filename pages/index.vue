<template>
  <div class="app">
    <div id="bkg"></div>
    <div v-if="user.isLoggedIn">
      <div id="header">
        <div id="logo">
          <div class="appName">Goby</div>
        </div>

        <div id="avatar">
          <img :src="user.photoURL" class="avatar" />

          <div style="display: none">
            Last Login: {{ currentUserLastLogin }} <button @click="logout()">Logout</button>
          </div>
        </div>
      </div>
      <br />

      <div id="carrousel" class="flex-container">
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>1</div>
        <div>2</div>
        <div>LAST</div>
      </div>

      <p>A Flexible Layout must have a parent element with the <em>display</em> property set to <em>flex</em>.</p>

      <p>Direct child elements(s) of the flexible container automatically becomes flexible items.</p>

      <br />
      <ul>
        <li>
          <NewCategory></NewCategory>
        </li>
        <li v-for="category in list" :key="category.id">
          <Category :category="category"></Category>
          <ul>
            <li>
              <NewItem :category="category"></NewItem>
            </li>
            <li v-for="item in category.items" :key="item.id">
              <Item :category="category" :item="item"></Item>
            </li>
          </ul>
        </li>
      </ul>
      <br />
      <br />
      <br />
      <br />
    </div>

    <LoaderCss v-else-if="waitingForUSerData"></LoaderCss>

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

import LoaderCss from '@/components/loader.vue'
import newCategory from '@/components/newCategory.vue'
import Category from '@/components/Category.vue'
import newItem from '@/components/newItem.vue'
import Item from '@/components/Item.vue'

export default {
  components: { LoaderCss, newCategory, Category, newItem, Item },
  data: () => {
    return {
      list: null,
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
    }
  }
}
</script>

<style>
#bkg {
  position: absolute;
  left: 0;
  top: 0;
  width: 360px;
  height: 730px;
  z-index: -1;
  background-image: url('/img/yt1.jpg');
  background-repeat: no-repeat;
  background-size: 360px 730px;
  opacity: 0;
}

#header {
  display: flex;
  justify-content: space-between;
  height: 12vw;
}

#logo {
  display: flex;
  background-image: url('/img/goby.png');
  background-repeat: no-repeat;
  background-size: 8vw 8vw;
  margin-left: 3vw;
  margin-top: 2vw;
}

.appName {
  margin-top: 2vw;
  margin-left: 10vw;
  font-weight: bold;
  color: #0077f3;
}

#avatar {
  display: flex;
  width: 7vw !important;
  margin-top: 2vw;
  margin-right: 3vw;
}

.avatar {
  vertical-align: middle;
  width: 7vw;
  height: 7vw;
  border-radius: 50%;
}

#carrousel {
  overflow: scroll;
  width: 100vw !important;
}
.flex-container {
  display: flex;
}

.flex-container > div {
  background-color: #f1f1f1;
  margin: 10px;
  padding: 20px;
  font-size: 30px;
}
</style>
