<template>
  <div class="sidebar-main p-0">
    <Icon icon="ant-design:close-outlined" width="30" height="30" @click="closeSidebar"/>
  </div>
</template>

<script>
export default {
    name: 'Sidebar',
    emits: ['closeSidebar'],
    setup(props, ctx) {
      function closeSidebar() {
        ctx.emit('closeSidebar');
      }
      return {
        closeSidebar
      }
    }
}
</script>

<style lang="scss">
.sidebar-main {
    background: #D26568;
    width: 100%;
    height: 100%;
    color: white;
}
</style>