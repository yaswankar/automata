<template>
  <div class="flowchart-node" :style="nodeStyle" 
    @mousedown="handleMousedown"
    @mouseover="handleMouseOver"
    @mouseleave="handleMouseLeave"
    @dblclick="openInfoPane"
    v-bind:class="{selected: options.selected === id}">
    <div class="node-port node-input"
       @mousedown="inputMouseDown"
       @mouseup="inputMouseUp">
    </div>
    <div class="node-main">
      <div v-text="type" class="node-type"></div>
      <div v-text="label" class="node-label"></div>
    </div>
    <div class="node-port node-output" 
      @mousedown="outputMouseDown">
    </div>
    <div v-show="show.delete" class="node-delete">&times;</div>
  </div>
</template>

<script>
import {ref, computed} from 'vue';

export default {
  name: 'WorkflowNode',
  emits: ['nodeSelected', 'linkingStart', 'linkingStop', 'openInfoPane'],
  props: {
    id: {
      type: Number,
      default: 1000,
      validator(val) {
        return typeof val === 'number'
      }
    },
    x: {
      type: Number,
      default: 0,
      validator(val) {
        return typeof val === 'number'
      }
    },    
    y: {
      type: Number,
      default: 0,
      validator(val) {
        return typeof val === 'number'
      }
    },
    type: {
      type: String,
      default: 'Default'
    },
    label: {
      type: String,
      default: 'input name'
    },
    options: {
      type: Object,
      default() {
        return {
          centerX: 1024,
          scale: 1,
          centerY: 140,
        }
      }
    }
  },
  setup(props, ctx) {
    const show = ref({
      delete: false
    });
    
    //Computed
    const nodeStyle = computed(() => {
      return {
        top: props.options.centerY + props.y * props.options.scale + 'px', // remove: props.options.offsetTop + 
        left: props.options.centerX + props.x * props.options.scale + 'px', // remove: props.options.offsetLeft + 
        transform: `scale(${props.options.scale})`,
      }
    });

    //Methods
    function handleMousedown(e) {
      const target = e.target || e.srcElement;
      if (target.className.indexOf('node-input') < 0 && target.className.indexOf('node-output') < 0) {
        ctx.emit('nodeSelected', e);
      }
      e.preventDefault();
    };
    function handleMouseOver() {
      show.value.delete = true;
    };
    function handleMouseLeave() {
      show.value.delete = false;
    };
    function outputMouseDown(e) {
      ctx.emit('linkingStart')
      e.preventDefault();
    };
    function inputMouseDown(e) {
      e.preventDefault();
    };
    function inputMouseUp(e) {
      ctx.emit('linkingStop')
      e.preventDefault();
    };
    function openInfoPane() {
      ctx.emit('openInfoPane', props)
    }

    return {
      show,
      nodeStyle,
      handleMousedown,
      handleMouseOver,
      handleMouseLeave,
      outputMouseDown,
      inputMouseDown,
      inputMouseUp,
      openInfoPane
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
$themeColor: #716AF5;
$portSize: 12;
.flowchart-node {
  margin: 0;
  width: 150px;
  height: 80px;
  position: absolute;
  box-sizing: border-box;
  border: none;
  background: white;
  z-index: 1;
  opacity: .9;
  cursor: move;
  transform-origin: top left;
  .node-main {
    text-align: center;
    .node-type {
      background: $themeColor;
      color: white;
      font-size: 13px;
      padding: 6px;
    }
    .node-label {
      font-size: 13px;
    }
  }
  .node-port {
    position: absolute;
    width: 12px;
    height: 12px;
    border: 1px solid #ccc;
    border-radius: 100px;
    background: white;
    z-index: 100;
    &:hover {
      background: $themeColor;
      border: 1px solid $themeColor;
    }
  }
  .node-input {
    top: 50%;
    left: 0;
    transform: translateX(-8px);
  }
  .node-output {
    top: 50%;
    right: 0;
    transform: translateX(8px);
  }
  .node-delete {
    position: absolute;
    right: -6px;
    top: -8px;
    font-size: 9px;
    width: 14px;
    height: 16px;
    color: #716AF5;
    cursor: pointer;
    background: white;
    border: 1px solid #716AF5;
    border-radius: 100px;
    text-align: center;
    &:hover{
      background: $themeColor;
      color: white;
    }
  }
}
.selected {
  box-shadow: 0 0 0 2px $themeColor;
}
</style>