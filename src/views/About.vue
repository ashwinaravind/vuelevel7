<template>
  <div class="about">
    <h1>This is Server Information</h1>
    <div v-if="signedIn">
         <h2>{{info}}</h2>
      <!-- <amplify-sign-out></amplify-sign-out> -->
      <button @click="signOut">Sign Out</button>

    </div>
      <p>{{ signedIn }}</p>
  </div>
</template>

<script>
import { Auth } from 'aws-amplify'
import axios from 'axios'
import { AmplifyEventBus } from 'aws-amplify-vue';
export default {
  data() {
    return {
       info: '',
      login: '',
      password: ''
      // signedIn: false
    }
  },
  props: {
    msg: String,
  },
  created(){
    this.findUser();
    AmplifyEventBus.$on('authState', info => {
      if(info === "signedIn") {
        this.findUser();
      } else {
        this.$store.state.signedIn = false;
        // this.signedIn = false;
        this.$store.state.user = null;
      }
    });
  },
  computed: {
    signedIn(){
      return this.$store.state.signedIn;
    }
  },
  methods: {
    signIn(){
      Auth.signIn(this.login, this.password)
        .then(user =>{
            this.$store.state.signedIn = !!user;
            this.$store.state.user = user;
        } )
        .catch(err => console.log(err));
    },
    signOut() {
      Auth.signOut()
        .then(data =>{
          this.$store.state.signedIn = !!data;
        } )
        .catch(err => console.log(err));
        this.$router.push('/')
    },
    async findUser() {
      try {
        const user = await Auth.currentAuthenticatedUser();
        // this.signedIn = true;
        this.$store.state.signedIn = true;
        this.$store.state.user = user;
        console.log(user);
      } catch(err) {
        // this.signedIn = false;
        this.$store.state.signedIn = false;
        this.$store.state.user = null;
      }
    }
  }
  ,
    mounted() {
      if(this.$store.state.user) {
        const jwt = this.$store.state.user
        .getSignInUserSession()
        .getIdToken()
        .getJwtToken();
        const config = {
          headers: {
            authorization: jwt
          }
        }
        console.log(config)
        axios.get('https://ep7d7j8sq0.execute-api.us-east-1.amazonaws.com/PROD', config)
        .then(val => this.info = val)
        .catch(err => console.log(err))
      }
    }
  }
  </script>
