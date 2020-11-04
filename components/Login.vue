<template>
  <div id="login">
    <div id="loginlogo"></div>
    <div id="fbButton">
      <img
        src="/img/fb.png"
        id="loginbutton"
        v-if="!$parent.waitingForUSerData && !$parent.user.isLoggedIn"
        @click="login('facebook')"
      />
    </div>
  </div>
</template>

<script>
export default {
  props: {},
  data() {
    return {}
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
      this.$parent.login(providerType)
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
