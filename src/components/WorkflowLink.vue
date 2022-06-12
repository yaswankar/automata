<template>
  <g @mouseover="handleMouseOver"
    @mouseleave="handleMouseLeave">
    <path :d="dAttr" :style="pathStyle"></path>
    <a v-if="show.delete" @click="deleteLink">
      <text 
        text-anchor="middle" 
        :transform="arrowTransform"
        font-size="22">&times;</text>
    </a>
    <path v-else d="M -1 -1 L 0 1 L 1 -1 z"
      :style="arrowStyle"
      :transform="arrowTransform"></path>
  </g>
</template>

<script>
import {ref, computed} from 'vue';

export default {
  name: 'WorkflowNode',
  emits: ['deleteLink'],
  props: {
    // start point position [x, y]
    start: {
      type: Array,
      default() {
        return [0, 0]
      }
    },
    // end point position [x, y]
    end: {
      type: Array,
      default() {
        return [0, 0]
      }
    },
    id: Number,
  },
  setup(props, ctx) {
    const show = ref({
      delete: false
    });
    
    //Computed
    const pathStyle = computed(() => {
      return {
        stroke: '#716AF5',
        strokeWidth: 2.73205,
        fill: 'none',
      }
    });

    const arrowStyle = computed(() => {
      return {
        stroke: '#716AF5',
        strokeWidth: 5.73205,
        fill: 'none',
      }
    });

    const arrowTransform = computed(() => {
      const [arrowX, arrowY] = calculateCenterPoint();
      const degree = calculateRotation()
      return `translate(${arrowX}, ${arrowY}) rotate(${degree})`;
    });

    const dAttr = computed(() => {
      let cx = props.start[0], cy = props.start[1], ex = props.end[0], ey = props.end[1];
      let x1 = cx, y1 = cy + 50, x2 = ex, y2 = ey - 50;
      return `M ${cx}, ${cy} C ${x1}, ${y1}, ${x2}, ${y2}, ${ex}, ${ey}`;
    });
    
    //Methods
    function handleMouseOver() {
      if (props.id) {
        show.value.delete = true;
      }
    };
    function handleMouseLeave() {
      show.value.delete = false;
    };
    function calculateCenterPoint() {
      // caculate arrow position: the center point between start and end
      const dx = (props.end[0] - props.start[0]) / 2;
      const dy = (props.end[1] - props.start[1]) / 2;
      return [props.start[0] + dx, props.start[1] + dy];
    };
    function calculateRotation() {
      // caculate arrow rotation
      const angle = -Math.atan2(props.end[0] - props.start[0], props.end[1] - props.start[1]);
      const degree = angle * 180 / Math.PI;
      return degree < 0 ? degree + 360 : degree;
    };
    function deleteLink() {
      ctx.emit('deleteLink')
    };
    return {
      show,
      pathStyle,
      arrowStyle,
      arrowTransform,
      dAttr,
      handleMouseOver,
      handleMouseLeave,
      calculateCenterPoint,
      calculateRotation,
      deleteLink
    }
  },
}
</script>

<style scoped lang="scss">
g {
  cursor: pointer;
}
</style>