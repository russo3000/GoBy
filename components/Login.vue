<template>
  <div id="login">
    <div id="loginlogo"></div>
    <div id="fbButton">
      <img
        src="/img/fb.png"
        id="loginbutton"
        v-if="!$parent.waitingForUSerData && !user.isLoggedIn"
        @click="login('facebook')"
      />
    </div>
  </div>
</template>

<script>
export default {
  props: {
    user: { type: Object, required: true },
    fb: { type: Object, required: true }
  },
  data() {
    return {
      token: ''
    }
  },
  mounted() {
    const imageNameNumber = Math.floor(Math.random() * 42)
    const imageName = "url('/img/items/" + imageNameNumber + ".jpg')"
    const login = window.document.querySelector('#login')

    login.style.setProperty('background-image', imageName)
    login.style.setProperty('height', window.innerHeight + 'px')
    login.style.setProperty('background-size', '100% ' + window.innerHeight + 'px')
  },
  created() {},
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
          this.token = result.credential.accessToken
          this.$parent.user_friends = JSON.parse(this.getFacebookFriendsList()).data
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    getFacebookFriendsList() {
      const graphUrl = 'https://graph.facebook.com/me/friends?access_token=' + this.token + '&fields=name,id,picture'
      const xmlHttp = new XMLHttpRequest()
      xmlHttp.open('GET', graphUrl, false) // false for synchronous request
      xmlHttp.send(null)
      return xmlHttp.responseText
    }
  }
}
</script>

<style scoped>
#login {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  background-repeat: no-repeat;
  width: 100%;
}

#loginlogo {
  display: flex;
  background-image: url('/img/goby.png');
  background-repeat: no-repeat;
  width: 145px;
  height: 145px;
}

#loginbutton {
  display: flex;
  width: 100%;
}

#fbButton {
  position: absolute;
  bottom: 2vw;
  width: 96%;
}
</style>
