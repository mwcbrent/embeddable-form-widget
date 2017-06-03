<template>
  <div class="alert">
    <p v-if="msg">{{msg}}</p>
    <p v-else="!msg">Sorry. We have issues.</p>
  </div>
</template>

<script>
export default {
  name: 'alert',
  props: ['msg']
};
</script>

<style lang="scss" scoped>
  .alert {
    align-items: center;
    background-color: #f2dede;
    color: #a94442;
    display: flex;
    height: 50px;
    padding: 5px 10px;
    width: 100%;
  }
</style>
