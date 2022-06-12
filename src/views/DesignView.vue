<template>
  <div class="design-view">
    <div class="workflow-section">
      <div class="text-xl font-bold underline">Workflow Design</div>
      <div class="tool-wrapper">
        <select v-model="newNodeType">
          <option v-for="(item, index) in nodeCategory" :key="index" :value="index">{{item}}</option>
        </select>
        <input type="text" v-model="newNodeLabel" placeholder="Input node label">
        <button @click="addNode">ADD</button>
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
export default {
  name: 'app',
  components: {
    Workflow,
    Sidebar
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
      const newNodeType = ref(0);
      const newNodeLabel = ref('');
      const nodeCategory = ref([
        'start',
        'action',
        'script',
        'service',
        'trigger',
        'stop',
      ]);

      //Methods
      function canvasClick(e) {
        console.log('canvas Click, event:', e)
      };
      function addNode() {
        let maxID = Math.max(0, ...scene.value.nodes.map((link) => {
          return link.id
        }))
        scene.value.nodes.push({
          id: maxID + 1,
          x: -400,
          y: 50,
          type: nodeCategory.value[newNodeType.value],
          label: newNodeLabel.value ? newNodeLabel.value: `test${maxID + 1}`,
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
      newNodeType,
      newNodeLabel,
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