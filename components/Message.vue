<template>
  <div class="message">
    <div class="heading">
      <span class="username">{{username}}</span>
      <span class="date">{{formattedDate}}</span>
    </div>
    <span class="text">{{text}}</span>
  </div>
</template>

<script>
import moment from 'moment';

export default {
  props: {
    username: {
      type: String,
      required: true,
    },
    text: {
      type: String,
      required: true,
    },
    date: {
      type: [String, Date],
      required: true,
    }
  },
  computed: {
    formattedDate() {
      moment.locale('ru');
      return moment(this.date).calendar();
    }
  },
}
</script>

<style lang="scss" scoped>
span {
  color: #fff;
}
.message {
  padding: 12px;
  margin-bottom: 12px;
  &:hover {
    background: rgba(255, 255, 255, 0.1);
  }
}

.heading {
  display: flex;
  align-items: center;
}

.date {
  font-size: 13px;
  color: #999;
}

.username {
  font-weight: 600;
  display: block;
  margin-right: 6px;
}

.text {
  display: block;
  margin-top: 6px;
  overflow: hidden;
  white-space: pre-wrap;
  white-space: break-spaces;
  word-wrap: normal;
  overflow-wrap: anywhere;
}
</style>