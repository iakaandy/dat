<template>
    <div class="index">
        <h1>
            <img @click="clickMe" src="https://raw.githubusercontent.com/iview/iview/master/assets/logo.png">
        </h1>
        <login-form @loginClick="onLoginClick($event)" @goToRegisterFormClick="isLoginMode = false"
                    v-if="isLoginMode">
        </login-form>
    </div>
</template>
<script>
  import LoginForm from '../components/start'

  export default {
    components: {
      LoginForm
    },
    data () {
      return {
        isLoginMode: true
      }
    },
    props: [],
    methods: {
      clickMe: function () {
        this.$router.go(-1)
      },
      onLoginClick (event) {
        // console.log(JSON.stringify(event))
        this.$router.push({name: 'servers-page'})
      },
      onRegisterClick (event) {
        if (event.ok) {
          this.$router.push({name: 'servers-page'})
        }
      }
    }
  }
</script>

<style scoped>
    /* CSS */
    .fadein-enter-active, .fadein-leave-active {
        transition: opacity 0.25s ease-out;
    }

    .fadein-enter, .fadein-leave-to {
        opacity: 0;
    }
</style>

<style scoped>
    .index {
        width: 100%;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        text-align: center;
    }

    .index h1 {
        height: 150px;
    }

    .index h1 img {
        height: 100%;
    }

    .index h2 {
        color: #666;
        /* margin-bottom: 200px; */
    }

    .index h2 p {
        margin: 0 0 50px;
    }

    .index .ivu-row-flex {
        height: 100%;
    }
</style>
