<template>
  <Header />
  <div class="design-view">
    <div class="workflow-section" :style="computeWidth">
      <div class="tool-wrapper mb-5 mt-5 px-3">
        <div style="margin-bottom: 20px;"><i>Double click to add nodes</i></div>
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
        @openInfoPane="updateInfoPane"
        :height="800"/>
    </div>
    <div class="sidepane" :style="computeSidebarWidth">
      <sidebar :nodeData="nodeData" />
    </div>
  </div>
</template>

<script>
import {ref, computed} from 'vue';
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
    const showSidebar = ref(true);
    const nodeData = ref({
      id: '',
      label: '',
      type: ''
    });
    const scene = ref({
        centerX: 1024,
        centerY: 140,
        scale: 1,
        nodes: [
        ],
        links: [
        ]
      });
      const nodeCategory = ref({
        start: 1,
        webhook: 2,
        service: 3,
        trigger: 4,
        stop: 5,
      });
      //Computed
      const computeWidth = computed(() => {
          if(showSidebar.value) {
            return {
              width: 'calc(100% - 400px)'
            }
          } else return { width: '100%'}
      });
      const computeSidebarWidth = computed(() => {
        if(showSidebar.value) return { width: '400px'};
        else return { width: 0 };
      })
      //Methods
      function canvasClick(e) {
        console.log('canvas Click, event:', e)
      };
      function addNode(label) {
        const excludes = ['start', 'stop'];
        if(excludes.includes(label)) {
          const startNode = scene.value.nodes.find(node => node.type === 'start');
          const stopNode = scene.value.nodes.find(node => node.type === 'stop');
          let errorFlag = false;
          let error = '';
          if(startNode && label === 'start') {
            errorFlag = true;
            error = 'start';
          } else if(stopNode && label === 'stop') {
            errorFlag = true;
            error = 'stop';
          }
          if(errorFlag) {
            window.alert(`Workflow already has a ${error} node`);
            return;
          }
        }
        let maxID = Math.max(0, ...scene.value.nodes.map((link) => {
          return link.id
        }))
        scene.value.nodes.push({
          id: maxID + 1,
          x: -400,
          y: 50,
          type: label,
          label: `${label}${maxID + 1}`,
        })
      };
      function nodeClick(id) {
        // console.log('node click', id);
      };
      function nodeDelete(id) {
        // console.log('node delete', id);
      };
      function linkBreak(id) {
        // console.log('link break', id);
      };
      function linkAdded(link) {
        // console.log('new link added:', link);
      }
      function updateInfoPane(node) {
        nodeData.value = { ...node };
      }

    return {
      showSidebar,
      nodeData,
      scene,
      nodeCategory,
      computeWidth,
      computeSidebarWidth,
      canvasClick,
      addNode,
      nodeClick,
      nodeDelete,
      linkBreak,
      linkAdded,
      // toggleSidebar,
      updateInfoPane
    }
  },
}
</script>

<style lang="scss">
.design-view {
  display: flex;
  width: 100%;
  .workflow-section {
    // width: calc(100% - 400px);
      .tool-wrapper {
        position: relative;
    }
  }
  .sidepane {
    // width: 400px;
  }
}
</style>