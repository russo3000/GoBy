<template>
  <div class="app">
    <div id="bkg"></div>

    <Login v-if="!waitingForUSerData && !user.isLoggedIn" :user="user" :fb="fb"></Login>

    <LoaderCss v-if="waitingForUSerData"></LoaderCss>

    <div v-if="user.isLoggedIn" id="main">
      <div id="header">
        <div id="logo">
          <div class="appName">Goby</div>
        </div>

        <div id="settings">
          <div id="logout" v-if="loggingOut">
            <div style="display: none">Last Login: {{ currentUserLastLogin }}</div>
            <button @click="logout()">Logout</button>
          </div>
          <div @click="loggingOut = !loggingOut">
            <el-button type="primary" icon="el-icon-setting" size="mini"></el-button>
          </div>
        </div>
      </div>

      <div id="friends">
        <div @click="getMyPlaces()" class="friend selectedFriend" :id="user.providerData[0].uid">
          <img :src="photoURL" class="friend-avatar" />
          <br />
          <div class="friend-name">Me</div>
        </div>

        <div
          v-for="friend in user_friends"
          :key="friend.id"
          :id="friend.id"
          @click="getFriendsPlaces(friend.id)"
          class="friend"
        >
          <img :src="friend.picture.data.url" class="friend-avatar" />
          <br />
          <div class="friend-name">{{ friend.name }}</div>
        </div>
      </div>

      <ul>
        <li v-for="category in list" :key="category.id">
          <Category :category="category"></Category>
          <div class="carrousel flex-container">
            <div v-for="item in category.items" :key="item.id">
              <Item :category="category" :item="item"></Item>
            </div>
            <NewItem :category="category" v-if="!showingFriendsPlaces"></NewItem>
          </div>
        </li>
      </ul>
      <div v-if="!list"><center>There are no Categories to Show</center></div>

      <div id="headerNewCategory" v-if="!showingFriendsPlaces">
        <NewCategory></NewCategory>
      </div>
      <br />
      &nbsp;
    </div>
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
import Login from '@/components/Login.vue'
import newCategory from '@/components/newCategory.vue'
import Category from '@/components/Category.vue'
import newItem from '@/components/newItem.vue'
import Item from '@/components/Item.vue'

/*
user_friends: [
        {
          name: 'Jason Essebag',
          picture: {
            data: {
              url: 'https://miro.medium.com/fit/c/336/336/0*H3IJ5FkJ1ut-xekL.'
            }
          }
        }
      ]
*/
export default {
  components: { LoaderCss, Login, newCategory, Category, newItem, Item },
  data: () => {
    return {
      list: null,
      loggingOut: false,
      photoURL: '', // Placeholders for what fb will return
      user: {
        isLoggedIn: false,
        displayName: 'Not Logged In' // Placeholders for what google will return
      },
      user_friends: null,
      waitingForUSerData: false,
      showingFriendsPlaces: false,
      currentUserLastLogin: '',
      token: '',
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
  mounted() {},
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
          .doc(this.user.providerData[0].uid)
          .get(getOptions)
          .then((doc) => {
            this.currentUserLastLogin = doc.data().lastLogin
            this.photoURL = doc.data().userphotourl
            this.getList(this.user.providerData[0].uid)
            this.getFriendsList(this.user.providerData[0].uid)
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
    logout() {
      let currentdate = new Date().toLocaleString()
      this.firebaseDb
        .collection('login')
        .doc(this.user.providerData[0].uid)
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
            .doc(this.user.providerData[0].uid)
            .set({ list: this.list })
            .then(() => {
              this.waitingForUSerData = false
            })
        })
      }
    },

    getList(uid) {
      // Optionf to user Firebase Get command
      var getOptions = {
        source: 'default'
      }

      this.firebaseDb
        .collection('list')
        .doc(uid)
        .get(getOptions)
        .then((doc) => {
          if (doc.data()) {
            this.list = doc.data().list

            if (this.list) {
              this.list.map((category) => {
                category.addingANewItem = false
                category.editingACategory = false
                category.newItem = { name: '', editingAnItem: false }
              })
            } else {
              this.list = null
            }
          } else {
            this.list = null
          }
        })
        .catch(function (error) {
          console.log('Error getting cached document:', error)
        })
    },
    getFriendsList(uid) {
      this.waitingForUSerData = true
      // Option to user Firebase Get command
      const getOptions = {
        source: 'default'
      }

      this.firebaseDb
        .collection('friends')
        .doc(uid)
        .get(getOptions)
        .then((doc) => {
          this.user_friends = doc.data().friends

          if (this.user_friends) {
            this.waitingForUSerData = false
          } else {
            this.friends = null
          }
        })
        .catch(function (error) {
          console.log('Error getting cached document:', error)
        })
    },

    getFriendsPlaces(myFriendId) {
      this.showingFriendsPlaces = true
      this.getList(myFriendId)

      var selectedFriends = document.getElementsByClassName('selectedFriend')

      if (selectedFriends.length > 0) {
        selectedFriends[0].classList.remove('selectedFriend')
      }

      document.getElementById(myFriendId).classList.add('selectedFriend')
    },

    getMyPlaces() {
      this.showingFriendsPlaces = false
      this.getList(this.user.providerData[0].uid)

      var selectedFriends = document.getElementsByClassName('selectedFriend')

      if (selectedFriends.length > 0) {
        selectedFriends[0].classList.remove('selectedFriend')
      }

      document.getElementById(this.user.providerData[0].uid).classList.add('selectedFriend')
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
  /*background-image: url('/img/yt2.png');*/
  background-repeat: no-repeat;
  background-size: 360px 730px;
  opacity: 1;
  background-color: #282828;
}

#main {
  background-color: #282828;
  color: #fff;
  font-family: Arial, Helvetica, sans-serif;
  width: 100%;
}
#header {
  display: flex;
  justify-content: space-between;
  height: 12vw;
}

#settings {
  display: flex;
  margin-right: 2vw;
  margin-top: 2vw;
}

#logo {
  display: flex;
  background-image: url('/img/goby.png');
  background-repeat: no-repeat;
  background-size: 8vw 8vw;
  margin-left: 3vw;
  margin-top: 2vw;
}
#headerNewCategory {
  margin-left: 3vw;
}
.appName {
  margin-top: 2vw;
  margin-left: 10vw;
  font-weight: bold;
  color: #fff;
}

.avatar {
  vertical-align: middle;
  width: 8vw;
  height: 8vw;
  border-radius: 50%;
}

.selectedFriend {
  background-color: #26384f;
}

#friends {
  display: flex;
  justify-content: space-evenly;
  border-bottom: 1px solid #3d3d3d;
  border-top: 1px solid #3d3d3d;
  padding-left: 2vw;
  height: 23vw;
}
.friend {
  text-align: center;
  padding-left: 2vw;
  padding-right: 2vw;
}
.friend-name {
  font-size: 3vw;
  margin-top: 2vw;
  text-align: center;
  color: gray;
}
.friend-avatar {
  vertical-align: middle;
  width: 14vw;
  height: 14vw;
  border-radius: 50%;
  margin-top: 3vw;
  padding-left: 2vw;
  padding-right: 2vw;
}

ul {
  list-style: none;
  padding: 0;
}
#carrousel {
  overflow: scroll;
  width: 100vw !important;
}

.carrousel {
  overflow: scroll;
  width: 100vw !important;
}

.flex-container {
  display: flex;
}

.flex-container > div {
  background-color: #f1f1f1;
  margin: 2vw;
  padding: 12vw;
  font-size: 5vw;
  width: 100vw;
}

input {
  width: 10vw;
}

#logout {
  padding-top: 2vw;
  padding-right: 4vw;
}

.el-input {
  margin-top: 1vw;
  width: 70%;
}
</style>
