<template>
  <div class="timestamp">
    <h5>{{ timestamp }}</h5>
  </div>
</template>

<script>
export default {
  data() {
    return {
      timestamp: '',
    };
  },
  created() {
    setInterval(this.getNow, 1);
  },
  methods: {
    getNow() {
      const today = new Date();
      const date =
        today.getFullYear() + '-' + (today.getMonth() + 1) + '-' + today.getDate();
      const time = today.getHours() + ':' + today.getMinutes() + ':' + today.getSeconds();
      const dateTime = date + ' ' + time;
      this.timestamp = dateTime;
    },
  },
};
</script>
<style scoped>
.timestamp {
  float: right;
}
</style>
