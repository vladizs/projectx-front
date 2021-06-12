<template>
  <div class="user" :class="{ admin: user.userType === 'admin', banned: user.isBanned }">
      <span class="username">{{ user.username }}</span>
      <img
        v-if="user.userType === 'admin'"
        src="~/assets/img/crown.svg"
        alt=""
        class="admin-crown" />
      <div v-if="user && currentUser && user.userId === currentUser.userId" class="isyou">(ты)</div>
    </div>
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      required: true,
    },
    currentUser: {
      type: Object,
      required: true,
    }
  },
}
</script>

<style lang="scss" scoped>
.user {
  display: flex;
  align-items: center;
  border-radius: 6px;
  padding: 6px;
  &:hover {
    background: rgba(255, 255, 255, 0.1);
  }
  span.username {
    color: #fff;
    line-height: 24px;
  }
  .isyou {
    margin-left: 6px;
    color: #959595;
    line-height: 24px;
  }
  &.banned {
    span {
      color: #ce4d4d;
    }
  }
  &.admin {
    span {
      color: #f7b047;
    }
  }
  .admin-crown {
    width: 20px;
    height: 20px;
    margin-left: 6px;
  }
}
</style>