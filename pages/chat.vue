<template>
  <div class="chat">
    <transition name="error">
      <div class="error-msg" v-if="errorMsg">{{ errorMsg }}</div>
    </transition>
    <aside class="aside" id="aside">
      <img src="~/assets/img/logo.svg" alt="" class="logo">
      <h2>Список юзеров чата</h2>
      <Users :users="users" v-if="users" :currentUser="currentUser" />
      <div class="userslist" v-else>
        <div class="user"><span>Загрузка...</span></div>
      </div>
    </aside>
    <main class="main" id="main">
      <div class="chat-title">
        <span class="title">project {x} main chat</span>
        <span class="theme">П3-19Р салам, остальным соболезную</span>
      </div>
      <Chat :messages="messages" v-if="messages" :currentUser="currentUser" />
      <div class="chat-message-input">
        <input type="text" class="textbox" placeholder="Введи что-нибудь" v-model="messageInput" @keydown.enter="sendMessage">
        <img src="~/assets/img/send-msg.svg" alt="Отправить сообщение" class="send-msg-btn" @click="sendMessage">
      </div>
    </main>
  </div>
</template>

<script>
export default {
  data() {
    return {
      users: null,
      errorMsg: null,
      currentUser: null,
      messages: null,
      messageInput: null,
      showContext: true,
    }
  },
  async beforeMount(){
    this.openWS();
    this.$axios.$get('/auth', {
      headers: {
        'Authorization': `Bearer ${this.$cookies.get('token')}`
      }
    }).then(res => {
      this.currentUser = res.content;
    }).catch(err => {
      this.showError(err);
    });
    this.$axios.$get('/user', {
      headers: {
        'Authorization': `Bearer ${this.$cookies.get('token')}`
      }
    }).then(res => {
      this.users = res.content.users;
    }).catch(err => {
      this.showError(err);
    });
    await this.$axios.$get('/message/get', {
      headers: {
        'Authorization': `Bearer ${this.$cookies.get('token')}`
      }
    }).then(res => {
      this.messages = res.content.messages;
      if (this.messages.length > 1) this.messages.reverse();
    }).catch(err => {
      this.showError(err);
    });
  },
  async middleware({ $cookies, $axios, redirect }) {
    if ($cookies.get('token')) {
      await $axios.$get('/auth', { headers: {
        'Authorization': `Bearer ${$cookies.get('token')}`,
      }}).then(res => {
        return;
      }).catch(err => {
        $cookies.remove('token');
        redirect('/')
      })
    } else {
      redirect('/');
    }
  },
  methods: {
    sendMessage() {
      this.$axios.setHeader('Content-Type', 'application/x-www-form-urlencoded', [
        'post'
      ]);
      this.$axios.setHeader('Authorization', `Bearer ${this.$cookies.get('token')}`);
      this.$axios.$post('/message/send', {
        text: this.messageInput,
      }).catch(err => {
        this.showError(err);
      });
      this.messageInput = null
    },
    openWS() {
      let ws = new WebSocket('ws://195.2.73.205:8080');
      ws.onmessage = (e) => {
        if(e.data === '1') {
          return;
        }
        const data = JSON.parse(e.data)
        if(data.msg) {
          this.messages.push(data.msg[0]);
          if (this.messages.length > 40) {
            this.messages.splice(0, 1);
          }
        }
        if(data.banned) {
          const targetUser = this.users.filter(user => {
            return user.userId == data.banned;
          })[0];
          this.users[this.users.indexOf(targetUser)].isBanned = true;
        }
        if(data.unbanned) {
          const targetUser = this.users.filter(user => {
            return user.userId == data.unbanned;
          })[0];
          this.users[this.users.indexOf(targetUser)].isBanned = false;
        }
        if(data.msgDelete) {
          const targetMsg = this.messages.filter(msg => {
            return msg.messageId == data.msgDelete;
          })[0];
          this.messages.splice(this.messages.indexOf(targetMsg), 1);
        }
        if(data.newUser) {
          this.users.push(data.newUser[0]);
        }
      };

      ws.onclose = function(e) {
        setTimeout(function() {
          this.openWS();
        }, 1000);
      };

      ws.onerror = function(err) {
        ws.close();
      };
    },
    showError(errorMsg){
      this.errorMsg = errorMsg;
      setTimeout(() => {
        this.errorMsg = null;
      }, 5000)
    },
  }
}
</script>

<style lang="scss" scoped>
.chat {
  display: flex;
}
aside {
  background: #080809;
  flex: 0 380px;
  height: 100vh;
  padding: 24px 0px;
  img.logo {
    display: block;
    width: 120px;
    margin: 0 auto;
  }
  h2 {
    margin-top: 24px;
    margin-bottom: 12px;
    color: #fff;
    font-weight: 400;
    font-size: 16px;
  }
}

main.main {
  flex: 1 1;
  display: flex;
  flex-direction: column;
  max-height: 100vh;
  .chat-title {
    flex: 0 1;
    display: flex;
    align-items: center;
    width: 100%;
    background: #080809;
    padding: 12px;
    span.title {
      color: #fff;
      font-size: 24px;
      font-weight: 700;
    }
    span.theme {
      color: #959595;
      font-size: 14px;
      margin-left: 24px;
    }
  }
  .chat-message-input {
    position: relative;
    flex: 0 1;
    padding: 24px;
    .textbox {
      display: block;
      width: 100%;
      padding: 12px 18px;
      padding-right: 60px;
      background: none;
      border: 2px solid #fff;
      border-radius: 18px;
      color: #fff;
      line-height: 18px;
      font-size: 16px;
      outline: none;
    }
    .send-msg-btn {
      position: absolute;
      top: 36px;
      right: 42px;
      cursor: pointer;
    }
  }
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

.error-enter-active, .error-leave-active {
  transition: all .5s;
}

.error-enter, .error-leave-to {
  top: -20%;
}
</style>