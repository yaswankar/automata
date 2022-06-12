<template>
  <div class="flowchart-container" ref="root"
    @mousemove="handleMove" 
    @mouseup="handleUp"
    @mousedown="handleDown">
    <svg width="100%" :height="`${height}px`">
      <workflow-link
        v-for="(link, index) in lines" 
        v-bind.sync="link"
        :key="`link${index}`"
        @deleteLink="linkDelete(link.id)">
      </workflow-link>
    </svg>
    <workflow-node
      v-for="(node, index) in scene.nodes" 
      v-bind.sync="node"
      :key="`node${index}`"
      :options="nodeOptions"
      @linkingStart="linkingStart(node.id)"
      @linkingStop="linkingStop(node.id)"
      @nodeSelected="nodeSelected(node.id, $event)">
    </workflow-node>
  </div>
</template>

<script>
import {computed, ref, onMounted} from 'vue';
import {cloneDeep} from 'lodash';
import WorkflowLink from './WorkflowLink.vue';
import WorkflowNode from './WorkflowNode.vue';
import { getMousePosition } from '../assets/utility/position';
export default {
  name: 'Workflow',
  components: {
    WorkflowLink,
    WorkflowNode
  },
  emits: ['linkAdded', 'linkBreak', 'nodeClick', 'canvasClick', 'nodeDelete'],
  props: {
    scene: {
      type: Object,
      default() {
        return {
          centerX: 1024,
          scale: 1,
          centerY: 140,
          nodes: [],
          links: [],
        }
      }
    },
    height: {
      type: Number,
      default: 400,
    },
  },
  setup(props, ctx) {
    const root = ref(null);
    const action = ref({
        linking: false,
        dragging: false,
        scrolling: false,
        selected: 0,
    });
      const mouse = ref({
        x: 0,
        y: 0,
        lastX: 0,
        lastY: 0,
      });
      const draggingLink = ref(null);
      const rootDivOffset = ref({
        top: 0,
        left: 0
      });
      const nodeOptions = computed(() => {
        return {
          centerY: props.scene.centerY,
          centerX: props.scene.centerX,
          scale: props.scene.scale,
          offsetTop: rootDivOffset.value.top,
          offsetLeft: rootDivOffset.value.left,
          selected: action.value.selected,
        }
      });
      const lines = computed(() => {
          const lines = props.scene.links.map((link) => {
          const fromNode = findNodeWithID(link.from)
          const toNode = findNodeWithID(link.to)
          let x, y, cy, cx, ex, ey;
          x = props.scene.centerX + fromNode.x;
          y = props.scene.centerY + fromNode.y;
          [cx, cy] = getPortPosition('right', x, y);
          x = props.scene.centerX + toNode.x;
          y = props.scene.centerY + toNode.y;
          [ex, ey] = getPortPosition('left', x, y);
          return { 
            start: [cx, cy], 
            end: [ex, ey],
            id: link.id,
          };
        })
        if (draggingLink.value) {
          let x, y, cy, cx;
          const fromNode = findNodeWithID(draggingLink.value.from)
          x = props.scene.centerX + fromNode.x;
          y = props.scene.centerY + fromNode.y;
          [cx, cy] = getPortPosition('right', x, y);
          // push temp dragging link, mouse cursor postion = link end postion 
          lines.push({ 
            start: [cx, cy], 
            end: [draggingLink.value.mx, draggingLink.value.my],
          })
        }
        return lines;
      });

    function findNodeWithID(id) {
      return props.scene.nodes.find((item) => {
          return id === item.id
      })
    };

    function getPortPosition(type, x, y) {
      if (type === 'left') {
        return [x, y+40];
      }
      else if (type === 'right') {
        return [x + 150, y + 40];
      }
    };

    function linkingStart(index) {
      action.value.linking = true;
      draggingLink.value = {
        from: index,
        mx: 0,
        my: 0,
      };
    }

    function linkingStop(index) {
      // add new Link
      if (draggingLink.value && draggingLink.value.from !== index) {
        // check link existence
        const existed = props.scene.links.find((link) => {
          return link.from === draggingLink.value.from && link.to === index;
        })
        if (!existed) {
          let maxID = Math.max(0, ...props.scene.links.map((link) => {
            return link.id
          }))
          const newLink = {
            id: maxID + 1,
            from: draggingLink.value.from,
            to: index,
          };
          props.scene.links.push(newLink)
          ctx.emit('linkAdded', newLink)
        }
      }
      draggingLink.value = null
    }

    function linkDelete(id) {
      const deletedLink = props.scene.links.find((item) => {
          return item.id === id;
      });
      if (deletedLink) {
        props.scene.links = props.scene.links.filter((item) => {
            return item.id !== id;
        });
        ctx.emit('linkBreak', deletedLink);
      }
    }

    function nodeSelected(id, e) {
      action.value.dragging = id;
      action.value.selected = id;
      ctx.emit('nodeClick', id);
      mouse.value.lastX = e.pageX || e.clientX + document.documentElement.scrollLeft
      mouse.value.lastY = e.pageY || e.clientY + document.documentElement.scrollTop
    }

    function handleMove(e) {
      if (action.value.linking) {
        [mouse.value.x, mouse.value.y] = getMousePosition(root.value, e);
        [draggingLink.value.mx, draggingLink.value.my] = [mouse.value.x, mouse.value.y];
      }
      if (action.value.dragging) {
        mouse.value.x = e.pageX || e.clientX + document.documentElement.scrollLeft
        mouse.value.y = e.pageY || e.clientY + document.documentElement.scrollTop
        let diffX = mouse.value.x - mouse.value.lastX;
        let diffY = mouse.value.y - mouse.value.lastY;
        mouse.value.lastX = mouse.value.x;
        mouse.value.lastY = mouse.value.y;
        moveSelectedNode(diffX, diffY);
      }
      if (action.value.scrolling) {
        [mouse.value.x, mouse.value.y] = getMousePosition(root.value, e);
        let diffX = mouse.value.x - mouse.value.lastX;
        let diffY = mouse.value.y - mouse.value.lastY;
        mouse.value.lastX = mouse.value.x;
        mouse.value.lastY = mouse.value.y;
        props.scene.centerX += diffX;
        props.scene.centerY += diffY;
        // this.hasDragged = true
      }
    };

    function handleUp(e) {
      const target = e.target || e.srcElement;
      if (root.value.contains(target)) {
        if (typeof target.className !== 'string' || target.className.indexOf('node-input') < 0) {
          draggingLink.value = null;
        }
        if (typeof target.className === 'string' && target.className.indexOf('node-delete') > -1) {
          this.nodeDelete(action.value.dragging);
        }
      }
      action.value.linking = false;
      action.value.dragging = null;
      action.value.scrolling = false;
    }

    function handleDown(e) {
      const target = e.target || e.srcElement;
      if ((target === root.value || target.matches('svg, svg *')) && e.which === 1) {
        action.value.scrolling = true;
        [mouse.value.lastX, mouse.value.lastY] = getMousePosition(root.value, e);
        action.value.selected = null; // deselectAll
      }
      ctx.emit('canvasClick', e);
    }

    function moveSelectedNode(dx, dy) {
      let index = props.scene.nodes.findIndex((item) => {
        return item.id === action.value.dragging
      })
      let left = props.scene.nodes[index].x + dx / props.scene.scale;
      let top = props.scene.nodes[index].y + dy / props.scene.scale;
      props.scene.nodes[index] = Object.assign(props.scene.nodes[index], {
        x: left,
        y: top,
      });
      props.scene.nodes = cloneDeep(props.scene.nodes);
    }

    function nodeDelete(id) {
      props.scene.nodes = props.scene.nodes.filter((node) => {
        return node.id !== id;
      })
      props.scene.links = props.scene.links.filter((link) => {
        return link.from !== id && link.to !== id
      })
      ctx.emit('nodeDelete', id)
    };
    onMounted(() => {
      rootDivOffset.value.top = root.value ? root.value.offsetTop : 0;
      rootDivOffset.value.left = root.value ? root.value.offsetLeft : 0;
    });

    return {
      root,
      action,
      mouse,
      draggingLink,
      rootDivOffset,
      nodeOptions,
      lines,
      findNodeWithID,
      getPortPosition,
      linkingStart,
      linkingStop,
      linkDelete,
      nodeSelected,
      handleMove,
      handleUp,
      handleDown,
      moveSelectedNode,
      nodeDelete
    };
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.flowchart-container {
  margin: 0;
  background: #ddd;
  position: relative;
  overflow: hidden;
  svg {
    cursor: grab;
  }
}
</style>
