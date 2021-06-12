<template>
  <div id="homepage">
    <transition name="error">
      <div class="error-msg" v-if="errorMsg">{{ errorMsg }}</div>
    </transition>
    <div class="main-form">
      <img src="~/assets/img/logo.svg" alt="">
      <transition name="appear">
        <a href="#" class="notmyacc" @click.prevent="stage = 'init'" v-if="stage !== 'init'">Подождите, хочу поменять ник</a>
      </transition>
      <p class="title">Введи свой ник</p>
      <span class="input-container">
        <input type="text" class="input nickname-input" placeholder="No Name" v-model="username" id="username" autocomplete="off" :disabled="stage !== 'init'" @keydown.enter="actionBtn">
      </span>
      <transition-group name="appear" class="group">
        <p class="title" v-if="stage == 'login'" :key="'loginText'" id="login-pass">Аккаунт с таким ником есть. Введи пароль от аккаунта</p>
        <p class="title" v-if="stage == 'register'" :key="'loginText'" id="register-pass">Придумай пароль</p>
        <input type="password" class="input password-input" id="password" v-if="stage !== 'init'" :key="'passwordInput'" v-model="password" @keydown.enter="actionBtn">
      </transition-group>
      <button class="btn" @click="actionBtn" :class="{disabled: blockBtn}">Далее</button>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      username: null,
      blockBtn: false,
      stage: 'init',
      errorMsg: null,
      password: null
    }
  },
  middleware({ $cookies, $axios, redirect }) {
    if ($cookies.get('token')) {
      $axios.setHeader('Authorization', `Bearer ${$cookies.get('token')}`)
      return $axios.$get('/auth').then(res => {
        redirect('/chat');
      })
    }
  },
  methods: {
    actionBtn(e) {
      if (this.blockBtn) {
        e.preventDefault();
        return;
      }
      this.stage === 'init' ? this.findUser() :
        this.stage === 'register' ? this.register() : this.login()
    },
    findUser() {
      if(!this.usernameValid()) return;
      this.blockBtn = true;
      this.$axios.$get('/auth/exists', {
        params: {
          username: this.username,
        },
      }).then(res => {
        this.blockBtn = false;
        if(res.content.exists) {
          this.stage = 'login';
          return;
        }
        this.stage = 'register';
      }).catch(err => {
        this.blockBtn = false;
        this.showError(err);
      })
    },
    register() {
      if(!this.passwordValid()) return;
      this.blockBtn = true;
      this.$axios.setHeader('Content-Type', 'application/x-www-form-urlencoded', [
        'post'
      ]);
      this.$axios.$post('/auth/register', {
        username: this.username,
        password: this.password,
      }).then(res => {
        this.blockBtn = false;
        this.$cookies.set('token', res.content.token);
        this.$router.push('/chat');
      }).catch(err => {
        this.blockBtn = false;
        this.showError(err)
      });
    },
    login() {
      if(!this.passwordValid()) return;
      this.blockBtn = true;
      this.$axios.$post('/auth/login', { 
        username: this.username,
        password: this.password,
       }).then(res => {
        this.blockBtn = false;
        this.$cookies.set('token', res.content.token);
        this.$router.push('/chat');
      }).catch(err => {
        this.blockBtn = false;
        if (err.response) {
          if (err.response.status == 401) {
            this.showError('Неверный пароль!');
          }
        }
        this.showError(err)
      });
    },
    showError(errorMsg){
      this.errorMsg = errorMsg;
      setTimeout(() => {
        this.errorMsg = null;
      }, 5000)
    },
    usernameValid() {
      if (!this.username || !/.{3,28}/ig.test(this.username)) {
        this.showError('Ник должен быть длиной от 3 до 28 символов')
        return false;
      }
      if(/[^\x00-\x7F]+/ig.test(this.username)) {
        this.showError('Ник может содержать только буквы английского алфавита, цифры и спец. символы');
        return false;
      }
      return true;
    },
    passwordValid() {
      if (!this.password || !/.{8,60}/ig.test(this.password)) {
        this.showError('Пароль должен быть длиной от 8 до 60 символов')
        return false;
      }
      return true;
    }
  },
}
</script>
<style lang="scss" scoped>
.main-form {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 360px;
  margin: 0 auto;
  margin-top: 200px;
}
p, a {
  font-size: 18px;
  font-weight: 500;
  text-decoration: none;
  color: #fff;
}
input.input {
  outline: none;
  background: transparent;
  border: none;
  border-bottom: 2px solid #fff;
  font-size: 18px;
  color: #fff;
  text-align: center;
  line-height: 30px;
  &:disabled {
    color: #888;
    border-color: #888;
  }
}
p.title {
  text-align: center;
  margin-top: 36px;
  margin-bottom: 12px;
}
.btn {
  background: #fff;
  font-size: 18px;
  font-weight: 500;
  border: none;
  margin-top: 48px;
  margin-bottom: 60px;
  padding: 12px 24px;
  border-radius: 6px;
  cursor: pointer;
  &.disabled {
    background: #555;
    cursor: not-allowed;
  }
}

.input-container {
  position: relative;
}

.group {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.error-msg {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: auto;
  padding: 12px 24px;
  text-align: center;
  background: #c52d2d;
  color: #fff;
}

a.notmyacc {
  font-size: 14px;
  margin-top: 18px;
  color: #ffd84a;
}

.error-enter-active, .error-leave-active {
  transition: all .5s;
}

.error-enter, .error-leave-to {
  top: -20%;
}

.appear-enter-active, .appear-leave-active {
  transition: all .5s;
}
.appear-enter, .appear-leave-to {
  opacity: 0;
}
</style>