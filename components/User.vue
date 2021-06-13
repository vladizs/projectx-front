<template>
  <div class="user" :class="{ admin: user.userType === 'admin', banned: user.isBanned }">
      <span class="username">{{ user.username }}</span>
      <img
        v-if="user.userType === 'admin'"
        src="~/assets/img/crown.svg"
        title="Гигант мысли (админ)"
        class="admin-crown" />
      <img v-if="user.isBanned" src="~/assets/img/banned.svg" class="banned-icon" title="Заблокирован чат">
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
  .admin-crown, .banned-icon {
    width: 20px;
    height: 20px;
    margin-left: 6px;
  }
}
</style>