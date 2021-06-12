<template>
  <div class="userslist" v-if="users">
    <div class="context-menu" v-if="contextMenu.shown" v-click-outside="closeContext" :style="{top: contextMenu.top+'px', left: contextMenu.left+'px'}">
      <div class="admin" v-if="contextMenu.rights === 'admin'">
        <div class="menu-action danger" v-if="!contextMenu.user.isBanned" @click="banUser(contextMenu.user)">
          <img src="~/assets/img/ban-icon.svg" alt="" class="icon">
          <span>Заблокировать чат</span>
        </div>
        <div class="menu-action danger" v-else @click="unbanUser(contextMenu.user)">
          <img src="~/assets/img/ban-icon.svg" alt="" class="icon" >
          <span>Разблокировать чат</span>
        </div>
      </div>
      <div class="user">
        <div class="menu-action">
          <img src="~/assets/img/id-icon.svg" alt="" class="icon">
          <span>Копировать userId</span>
        </div>
      </div>
      <div class="self" v-if="contextMenu.self" @click="logout">
        <div class="menu-action danger">
          <img src="~/assets/img/logout-icon.svg" alt="" class="icon">
          <span>Выйти из аккаунта</span>
        </div>
      </div>
    </div>
    <User v-for="user in users" :key="user.userId" :user="user" :currentUser="currentUser" @contextmenu.native.prevent="openContext($event, user)"/>
  </div>
</template>

<script>
export default {
  props: {
    users: {
      type: Array,
      required: true
    },
    currentUser: {
      type: Object,
      required: true,
    }
  },
  data() {
    return {
      contextMenu: {
        shown: false,
        top: 0,
        left: 0,
        rights: 'user',
        self: false,
        user: null,
      },
    }
  },
  methods: {
    closeContext() {
      this.contextMenu.shown = false;
    },
    openContext(e, user) {
      if (this.currentUser.userId !== user.userId){
        this.contextMenu.self = false;
        if(this.currentUser.userType !== 'admin') return;
      } else {
        this.contextMenu.self = true;
      }
      if(this.currentUser.userType !== 'admin') {
        this.contextMenu.rights = 'user';
      } else {
        this.contextMenu.rights = 'admin';
      }
      this.contextMenu.user = user;
      this.contextMenu.top = e.clientY;
      this.contextMenu.left = e.clientX+1;
      this.contextMenu.shown = true;
    },
    banUser(user) {
      this.$axios.setHeader('Content-Type', 'application/x-www-form-urlencoded', [
        'post'
      ]);
      this.$axios.setHeader('Authorization', `Bearer ${this.$cookies.get('token')}`);
      this.$axios.$post('/user/ban', {
        userId: user.userId,
      });
      this.closeContext();
    },
    unbanUser(user) {
      this.$axios.setHeader('Content-Type', 'application/x-www-form-urlencoded', [
        'post'
      ]);
      this.$axios.setHeader('Authorization', `Bearer ${this.$cookies.get('token')}`);
      this.$axios.$post('/user/unban', {
        userId: user.userId,
      });
      this.closeContext();
    },
    logout() {
      this.$cookies.remove('token');
      this.$router.push('/');
    }
  }
};
</script>

<style lang="scss" scoped>
.userslist {
  overflow-y: auto;
  max-height: 400px;
  height: 50%;
  &::-webkit-scrollbar {
    width: 12px;
    &-track {
      border: 1px solid #fff;
      border-radius: 10px;
    }
    &-thumb {
      border-radius: 10px;
      background: #fff;
      border: 1px solid #fff;
    }
  }
}
.context-menu {
  position: absolute;
  width: 300px;
  background: #212226;
  border-radius: 6px;
  z-index: 100;
  overflow: hidden;
  .menu-action {
    display: flex;
    align-items: center;
    font-size: 13px;
    line-height: 24px;
    padding: 6px 12px;
    cursor: pointer;
    &:hover {
      background: rgba(255, 255, 255, 0.1);
    }
    img {
      margin-right: 6px;
      width: 18px;
      height: 18px;
    }
    span {
      color: #fff;
    }
    &.danger {
      span{
        color: #ff4f4f;
      }
    }
  }
}
</style>
