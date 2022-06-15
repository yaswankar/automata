<template>
  <div class="sidebar-main p-0">
    <div>{{getLabel}}</div>
  </div>
</template>

<script>
import {computed} from 'vue';
export default {
    name: 'Sidebar',
    emits: ['closeSidebar'],
    props: ['nodeData'],
    setup(props, ctx) {
      const getLabel = computed(() => {
        console.log(props?.nodeData?.label);
        return props?.nodeData?.label || '';
      })
      function closeSidebar() {
        ctx.emit('closeSidebar');
      }
      return {
        getLabel,
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