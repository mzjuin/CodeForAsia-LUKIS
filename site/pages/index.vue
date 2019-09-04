<template>
  <div class="container table-responsive">
    <div v-if="!user" id="google-signin-btn" />
    <div v-else>
      <table class="table table-striped table-bordered">
        <thead class="thead-dark">
          <tr>
            <th v-for="k in Object.keys(exercises[0])" :key="k">
              {{ k }}
            </th>
            <th>Enter</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(e, index) in exercises" :key="index">
            <th v-for="v in e" :key="v">{{ v }}</th>
            <th>
              <nuxt-link :to="'/exercises/' + e.name.toLowerCase()"
                ><button type="button" class="btn btn-primary">
                  Enter
                </button></nuxt-link
              >
            </th>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
// import _ from 'lodash'
// import Vue from 'vue'
import { mapGetters, mapMutations } from 'vuex'

export default {
  data() {
    return {
      exercises: [
        {
          name: 'Alphabet',
          total: 26
        },
        {
          name: 'Digit',
          total: 10
        },
        {
          name: 'Animals',
          total: 100
        }
      ]
    }
  },
  computed: {
    ...mapGetters(['user'])
  },
  mounted() {
    gapi.signin2.render('google-signin-btn', {
      onsuccess: this.onSignIn // note, no "()" here
    })
  },
  methods: {
    ...mapMutations(['SETUSER']),
    onSignIn(user) {
      debugger
      // do stuff, for example
      const email = user.getBasicProfile().getEmail()
      const displayName = user.getBasicProfile().getName()
      this.SETUSER({ email, displayName })
      // const token = user.Zi.access_token
      // window.localStorage.setItem('fb-token', token)
      // const tempURL = window.localStorage.getItem('temp-url')
      // if (tempURL) {
      //   setTimeout(() => {
      //     this.$router.push(tempURL)
      //   }, 0)
      // }
    }
  }
}
</script>

<style scoped>
.login-container {
  margin-top: 150px;
  width: 100%;
  display: flex;
  justify-content: center;
}
.login-tile {
  width: 400px;
  background: #ffffff;
  border-radius: 1px;
}
.logo {
  margin-bottom: 12px;
}
.login-header {
  font-family: 'IBM Plex Sans';
  font-weight: 600;
  font-size: 28px;
  color: #152935;
  line-height: 35px;
  margin-bottom: 24px;
}
</style>
