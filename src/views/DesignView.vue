<template>
  <Header />
  <div class="design-view">
    <div class="workflow-section">
      <div class="tool-wrapper mb-5 mt-5 px-3">
        <div class="flex flex-row">
          <Icon icon="carbon:play" @dblclick="addNode('start')" class="basis-1/5" width="50" height="50" />
          <Icon icon="carbon:webhook" @dblclick="addNode('webhook')" class="basis-1/5" width="50" height="50" />
          <Icon icon="carbon:flash" @dblclick="addNode('trigger')" class="basis-1/5" width="50" height="50" />
          <Icon icon="carbon:cloud-services" @dblclick="addNode('service')" class="basis-1/5" width="50" height="50" />
          <Icon icon="carbon:stop" @dblclick="addNode('stop')" class="basis-1/5" width="50" height="50" />
        </div>
      </div>
      <workflow :scene="scene" 
        @nodeClick="nodeClick"
        @nodeDelete="nodeDelete"
        @linkBreak="linkBreak"
        @linkAdded="linkAdded"
        @canvasClick="canvasClick"
        :height="800"/>
    </div>
    <div class="sidepane">
      <sidebar />
    </div>
  </div>
</template>

<script>
import {ref} from 'vue';
import Workflow from '../components/Workflow.vue'
import Sidebar from '../components/Sidebar.vue'
import Header from '../components/Header.vue'
export default {
  name: 'app',
  components: {
    Workflow,
    Sidebar,
    Header
  },
  setup() {
    const scene = ref({
        centerX: 1024,
        centerY: 140,
        scale: 1,
        nodes: [
        ],
        links: [
        ]
      });
      const nodeCategory = ref([
        'start',
        'webhook',
        'service',
        'trigger',
        'stop',
      ]);

      //Methods
      function canvasClick(e) {
        console.log('canvas Click, event:', e)
      };
      function addNode(label) {
        let maxID = Math.max(0, ...scene.value.nodes.map((link) => {
          return link.id
        }))
        scene.value.nodes.push({
          id: maxID + 1,
          x: -400,
          y: 50,
          type: nodeCategory.value[label],
          label: `${label}${maxID + 1}`,
        })
      };
      function nodeClick(id) {
        console.log('node click', id);
      };
      function nodeDelete(id) {
        console.log('node delete', id);
      };
      function linkBreak(id) {
        console.log('link break', id);
      };
      function linkAdded(link) {
        console.log('new link added:', link);
      }

    return {
      scene,
      nodeCategory,
      canvasClick,
      addNode,
      nodeClick,
      nodeDelete,
      linkBreak,
      linkAdded
    }
  },
}
</script>

<style lang="scss">
.design-view {
  display: flex;
  width: 100%;
  .workflow-section {
    width: calc(100% - 300px);
      .tool-wrapper {
        position: relative;
    }
  }
  .sidepane {
    width: 300px;
  }
}
</style>