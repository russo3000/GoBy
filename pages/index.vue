<template>
  <div class="app">
    <div v-if="user.isLoggedIn">
      <img :src="user.photoURL" class="avatar" />

      <div>Last Login: {{ currentUserLastLogin }}</div>
      <button @click="logout()">Logout</button>
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
import Task from '@/components/Task.vue'
import Presentation from '@/components/Presentation.vue'

import firebase from 'firebase/app'
import 'firebase/auth'
import 'firebase/firestore'

// When a user logs in or out, save that in the store

export default {
  components: {
    Task,
    Presentation
  },
  data: () => {
    return {
      counter: 0,
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

    //Here is the Firebase Authentication Magic happening returning a FireBase authenticated User
    this.fb.auth().onAuthStateChanged((fbuser) => {
      if (fbuser != null) {
        this.user = fbuser

        this.user.isLoggedIn = true
        var currentdate = new Date().toLocaleString()

        this.firebaseDb
          .collection('login')
          .doc(this.user.uid)
          .set({ lastLogin: this.user.displayName + ' : ' + currentdate })
          .then(() => {
            console.log('Last Login successfully written to firebase')
          })

        // Optionf to user Firebase Get command
        var getOptions = {
          source: 'default'
        }

        this.firebaseDb
          .collection('login')
          .doc(this.user.uid)
          .get(getOptions)
          .then((doc) => {
            this.currentUserLastLogin = doc.data().lastLogin
          })
          .catch(function (error) {
            console.log('Error getting cached document:', error)
          })

        this.waitingForUSerData = false
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
        .then((res) => {
          this.user = res.user
          this.user.isLoggedIn = true
        })
        .catch(function (error) {
          const errorCode = error.code
          const errorMessage = error.message
          const email = error.email
          const credential = error.credential
          console.log(errorCode, errorMessage, email, credential)
        })
    },
    logout() {
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
    }
  }
}
</script>

<style>
.lds-facebook {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
}
.lds-facebook div {
  display: inline-block;
  position: absolute;
  left: 8px;
  width: 16px;
  background: #fff;
  animation: lds-facebook 1.2s cubic-bezier(0, 0.5, 0.5, 1) infinite;
}
.lds-facebook div:nth-child(1) {
  left: 8px;
  animation-delay: -0.24s;
}
.lds-facebook div:nth-child(2) {
  left: 32px;
  animation-delay: -0.12s;
}
.lds-facebook div:nth-child(3) {
  left: 56px;
  animation-delay: 0;
}
@keyframes lds-facebook {
  0% {
    top: 8px;
    height: 64px;
  }
  50%,
  100% {
    top: 24px;
    height: 32px;
  }
}
</style>
