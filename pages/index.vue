<template>
  <v-card tile class="elevation-10">
    <v-card-text class="layout column justify-center">
      <div class="text-xs-center hidden-md-and-up">
        <img src="../assets/logo1-alpha.png" alt="Guideme" height="60px">
        <h1 class="headline primary--text">{{apptitle}}</h1>
        <div class="body-2 grey--text">Agile Tracker.</div>
      </div>
      <div class="display-1 primary--text text-xs-center">Sign <u>in</u></div>
    </v-card-text>
    <v-divider></v-divider>
    <v-card-text class="layout justify-center">
      <div class="caption text-xs-center">
        <b>Sign In</b> below to continue managing your projects.
        <hr class="primary my-1">
      </div>
      <v-alert
        v-if="!!appError"
        :value="!!appError"
        type="error"
        dismissible
        @input="clearError"
      >
        {{appError.message}}
      </v-alert>
    </v-card-text>
    <v-layout row wrap justify-center>
      <v-form class="flex px-2"
        v-model="valid" 
        @submit.prevent="signin"
        @keydown.prevent.enter>
        <v-text-field class="my-0"
          label="Email or User Name"
          name="emailOrUsername"
          v-model="newUser.emailOrUsername"
          type="text"
          required
          :rules="[v => !!v || 'Field is required']"/>
        <v-text-field class="my-0"
          label="Password"
          name="password"
          v-model="newUser.password"
          type="password"
          required
          :rules="[v => !!v || 'Field is required']"/>
        <v-btn color="success" block type="submit" :disabled="!valid">Sign in</v-btn>
      </v-form>
    </v-layout>
    <hr class="primary my-1" v-show="!isLoading">
    <v-progress-linear
      color="info"
      height="2"
      v-show="!!isLoading"  
      :indeterminate="true"/>
    <v-card-actions>
      <em>Not Registered yet?</em>
      <v-spacer></v-spacer>
      <v-btn color="primary" small nuxt to="/signup">Sign up</v-btn>
    </v-card-actions>
  </v-card>
</template>
<script>
import { mapState, mapMutations } from 'vuex'
import { routeMixin } from '@/mixins'
export default {
  layout: 'login',
  mixins: [routeMixin],
  data: () => ({
    valid: false,
    newUser: {
      emailOrUsername: '',
      password: ''
    },
    isLoading: false
  }),
  computed: mapState([
    'appLoading',
    'appError'
  ]),
  methods: {
    ...mapMutations([
      'setError',
      'clearError'
    ]),
    signin () {
      this.isLoading = true
      if (this.valid) {
        const emailOrUsername = this.newUser.emailOrUsername.toLowerCase()
        // const emailPattern = new RegExp(/[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,3}$/)
        // const payload = {
        //   email: emailPattern.test(emailOrUsername)
        //     ? emailOrUsername
        //     : this.$store.getters.emailByUsername(emailOrUsername),
        //   username: !this.$store.getters.available(emailOrUsername)
        //     ? emailOrUsername
        //     : this.$store.getters.usernameByEmail(emailOrUsername),
        //   password: this.password
        // }

        const dummyPayload = {
          username: emailOrUsername,
          email: emailOrUsername + '.dummy@guidemy.io',
          displayName: emailOrUsername.toUpperCase()
        }
        // this.$store.dispatch('signin', payload)
        this.$store.dispatch('signup', dummyPayload)
          .then((result) => {
            if (result) {
              if (result.username) {
                // console.log(result)
                console.warn(`User ${result.username} - ${result.email}: logged On sucessfully`)
              } else throw new Error('Invalid E-mail or Username!!')
            } else throw Error('Request failed!!')
          })
          .then(() => this.$store.dispatch('fetchAppData'))
          .then(response => {
            // console.log(response)
            this.$router.push('/dashboard')
          })
          .catch(error => {
            console.warn(error)
            this.setError(error.message || error)
          })
      }
      this.isLoading = false
    }
  }
}
</script>
