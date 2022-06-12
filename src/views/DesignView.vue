<template>
  <div>
    <h1>Workflow Design</h1>
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
</template>

<script>
import {ref} from 'vue';
import Workflow from '../components/Workflow.vue'
export default {
  name: 'app',
  components: {
    Workflow
  },
  setup() {
    const scene = ref({
        centerX: 1024,
        centerY: 140,
        scale: 1,
        nodes: [
          {
            id: 2,
            x: -700,
            y: -69,
            type: 'Action',
            label: 'test1',
          },
          {
            id: 4,
            x: -357,
            y: 80,
            type: 'Script',
            label: 'test2',
          },
          {
            id: 6,
            x: -557,
            y: 80,
            type: 'Rule',
            label: 'test3',
          }
        ],
        links: [
          {
            id: 3,
            from: 2, // node id the link start
            to: 4,  // node id the link end
          }
        ]
      });
      const newNodeType = ref(0);
      const newNodeLabel = ref('');
      const nodeCategory = ref([
        'rule',
        'action',
        'script',
        'decision',
        'fork',
        'join',
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
  .tool-wrapper {
    position: relative;
  }
</style>