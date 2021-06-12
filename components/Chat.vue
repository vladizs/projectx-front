<template>
  <div class="chat-inner" ref="chatblock">
    <div class="context-menu" v-if="contextMenu.shown" v-click-outside="closeContext" :style="{top: contextMenu.top+'px', left: contextMenu.left+'px'}">
      <div class="admin" v-if="contextMenu.rights === 'admin'">
        <div class="menu-action danger" @click="deleteMsg(contextMenu.message)">
          <img src="~/assets/img/delete-icon.svg" alt="" class="icon">
          <span>Удалить сообщение</span>
        </div>
      </div>
      <div class="self" v-if="contextMenu.self && contextMenu.rights !== 'admin'" @click="deleteMsg(contextMenu.message)">
        <div class="menu-action danger">
          <img src="~/assets/img/delete-icon.svg" alt="" class="icon">
          <span>Удалить сообщение</span>
        </div>
      </div>
      <div class="self"></div>
    </div>
    <Message
      v-for="message in messages"
      :key="message.messageId"
      :username="message.sender.username"
      :text="message.text"
      :date="message.date"
      @contextmenu.native.prevent="openContext($event, message)"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      contextMenu: {
        shown: false,
        top: 0,
        left: 0,
        rights: 'user',
        self: false,
        message: null,
      },
    }
  },
  props: {
    messages: {
      type: Array,
      required: true
    },
    currentUser: {
      type: Object,
      required: true,
    }
  },
  updated() {
    this.scrollToBottom();
  },
  methods: {
    scrollToBottom() {
      const chatblock = this.$refs.chatblock;
      chatblock.scrollTop = chatblock.scrollHeight;
    },
    closeContext() {
      this.contextMenu.shown = false;
    },
    openContext(e, message) {
      if (this.currentUser.userId !== message.sender.userId){
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
      this.contextMenu.message = message;
      this.contextMenu.top = e.clientY+6;
      this.contextMenu.left = e.clientX+6;
      this.contextMenu.shown = true;
    },
    deleteMsg(message) {
      this.$axios.setHeader('Content-Type', 'application/x-www-form-urlencoded', [
        'post'
      ]);
      this.$axios.setHeader('Authorization', `Bearer ${this.$cookies.get('token')}`);
      this.$axios.$post('/message/delete', {
        messageId: message.messageId,
      });
      this.closeContext();
    },
  }
};
</script>

<style lang="scss" scoped>
.chat-inner {
  flex: 1 1;
  overflow-y: auto;
  min-height: min-content;
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
