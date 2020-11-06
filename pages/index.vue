<template>
  <div class="app">
    <div id="bkg"></div>

    <Login v-if="!waitingForUSerData && !user.isLoggedIn"></Login>
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
        <div @click="getMyPlaces()" class="friend selectedFriend" :id="user.uid">
          <img :src="user.photoURL" class="friend-avatar" />
          <br />
          <div class="friend-name">Me</div>
        </div>

        <div
          v-for="friend in user.user_friends"
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
        <li v-for="category in user.list" :key="category.id">
          <Category :category="category"></Category>
          <div class="carrousel flex-container">
            <div v-for="item in category.items" :key="item.id">
              <Item :category="category" :item="item"></Item>
            </div>
            <NewItem :category="category" v-if="!showingFriendsPlaces"></NewItem>
          </div>
        </li>
      </ul>
      <div v-if="!user.list"><center>There are no Categories to Show</center></div>

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

export default {
  components: { LoaderCss, Login, newCategory, Category, newItem, Item },
  data: () => {
    return {
      loggingOut: false,
      photoURL: '', // Placeholders for what fb will return
      user: {
        isLoggedIn: false,
        uid: '',
        displayName: 'Not Logged In', // Placeholders for what google will return
        photoURL: '', // Placeholders for what google will return
        user_friends: null,
        list: null
      },
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
    this.waitingForUSerData = true

    this.fb = firebase

    // Need this cuz of this strange bug i can't find any info about
    if (!this.fb.apps.length) {
      this.fb.initializeApp(this.firebaseConfig)
    }

    this.firebaseDb = this.fb.firestore()

    // Here is the Firebase Authentication Magic happening returning a FireBase authenticated User
    this.fb.auth().onAuthStateChanged((fbuser) => {
      if (fbuser) {
        //Got Loggedin user, maybe try getting if from the local storage by it's uid
        this.user.uid = localStorage.getItem('goby-user-uid')

        if (this.user.uid) {
          this.user = JSON.parse(localStorage.getItem(this.user.uid))
          if (!this.user.list) {
            this.getList(this.user.uid)
          }
        }

        this.waitingForUSerData = false
      } else {
        this.waitingForUSerData = false
        this.user.isLoggedIn = false
      }
    })
  },

  methods: {
    login(providerType) {
      let provider = null

      switch (providerType) {
        case 'facebook':
          provider = new this.fb.auth.FacebookAuthProvider().addScope('user_friends')
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
        .signInWithPopup(provider)
        .then((result) => {
          if (result.user != null) {
            this.token = result.credential.accessToken
            this.user.uid = result.additionalUserInfo.profile.id
            this.user.photoURL = result.additionalUserInfo.profile.picture.data.url
            this.user.isLoggedIn = true

            // Options to use Firebase Get command
            const getOptions = { source: 'default' }

            this.firebaseDb
              .collection('login')
              .doc(this.user.uid)
              .get(getOptions)
              .then((doc) => {
                this.currentUserLastLogin = doc.data().lastLogin

                this.getList(this.user.uid)
              })
              .catch(function (error) {
                console.log('Error getting cached document:', error)
              })
          } else {
            this.user.isLoggedIn = false
          }

          this.saveProfilePicture(
            result.additionalUserInfo.profile.picture.data.url,
            result.additionalUserInfo.profile.id
          )

          this.user.user_friends = JSON.parse(this.getFacebookFriendsList()).data
          this.saveFriendsList(this.user.user_friends, this.user.uid)

          localStorage.setItem('goby-user-uid', this.user.uid)
          localStorage.setItem(this.user.uid, JSON.stringify(this.user))
        })
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
      if (this.user.list) {
        this.user.list.map((category) => {
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
            .set({ list: this.user.list })
            .then(() => {
              this.waitingForUSerData = false
            })
        })
      }
    },

    getList(uid) {
      this.waitingForUSerData = true
      // Optionf to user Firebase Get command
      var getOptions = {
        source: 'default'
      }

      this.firebaseDb
        .collection('list')
        .doc(uid)
        .get(getOptions)
        .then((doc) => {
          this.user.list = doc.data().list

          if (this.user.list) {
            this.user.list.map((category) => {
              category.addingANewItem = false
              category.editingACategory = false
              category.newItem = { name: '', editingAnItem: false }
            })

            this.waitingForUSerData = false
          } else {
            this.user.list = null
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
          this.user.user_friends = doc.data().friends

          this.waitingForUSerData = false
        })
        .catch(function (error) {
          console.log('Error getting cached document:', error)
        })
    },

    getFacebookFriendsList() {
      const graphUrl = 'https://graph.facebook.com/me/friends?access_token=' + this.token + '&fields=name,id,picture'
      const xmlHttp = new XMLHttpRequest()
      xmlHttp.open('GET', graphUrl, false) // false for synchronous request
      xmlHttp.send(null)
      return xmlHttp.responseText
    },

    saveFriendsList(userFriends, uid) {
      this.firebaseDb
        .collection('friends')
        .doc(uid)
        .set({ friends: userFriends })
        .then(() => {})
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
      this.getList(this.user.uid)

      var selectedFriends = document.getElementsByClassName('selectedFriend')

      if (selectedFriends.length > 0) {
        selectedFriends[0].classList.remove('selectedFriend')
      }

      document.getElementById(this.user.uid).classList.add('selectedFriend')
    },

    saveProfilePicture(photoURL, uid) {
      this.firebaseDb = this.fb.firestore()

      this.firebaseDb
        .collection('login')
        .doc(uid)
        .set({ userphotourl: photoURL })
        .then(() => {})
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
  background-image: url('/img/yt1.jpg');
  background-repeat: no-repeat;
  background-size: 360px 730px;
  opacity: 1;
  background-color: #282828;
  top: 95px;
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
  margin-bottom: 5vw;
}

.flex-container {
  display: flex;
  width: 100%;
}

.flex-container > div {
  display: flex;

  width: 130px;
  height: 160px;
  margin-left: 4vw;
  margin-top: 2vw;
  font-size: 13px;
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
  width: 100%;
  margin-bottom: 2vw;
}
</style>
