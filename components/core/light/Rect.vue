<template lang="pug">
div LightRect
  CoreObject3D(v-bind="props", :object3d="light")
</template>
<script setup lang="ts">
import { RectAreaLight } from "three";
import type { RenderDataflow } from "~~/utils/RenderDataflow";
const props = withDefaults(
  defineProps<{
    dx?: number;
    dy?: number;
    dz?: number;
    rx?: number;
    ry?: number;
    rz?: number;
    sx?: number;
    sy?: number;
    sz?: number;
    strength?: number;
    color?: number;
    width?: number;
    height?: number;
  }>(),
  {
    strength: 1,
    color: 0xffffff,
    width: 1,
    height: 1,
  }
);
/* start render flow */
// get flow
let flow = inject<RenderDataflow<{}, {}> | null>("flow0", null);
let nestCount = 0;
for (var i = 0; flow; i++) {
  flow = inject<RenderDataflow<{}, {}> | null>("flow" + i, null);
  flow && (nestCount = i);
}
flow = inject<RenderDataflow<{}, {}> | null>("flow" + nestCount, null);
// new child flow
const childFlow = flow?.newChild({}, {});
// provide child flow
provide("flow" + (nestCount + 1), childFlow);
// get random id
const id = useRandomId();
// add this to callback
childFlow?.emit?.("addCallback", [id, 1]);
onUnmounted(() => {
  // remove this from callback
  childFlow?.emit?.("removeCallback", id);
});

watchEffect(() => {
  if (flow?.props.loadings[id] === -1) {
    // on load
    childFlow?.emit?.("updateLoadings", [id, 0]);
  }
});
watchEffect(() => {
  if (flow?.props.suspendings[id] === -1) {
    // on suspend
    childFlow?.emit?.("updateSuspendings", [id, 0]);
  }
});
watchEffect(() => {
  if (flow?.props.renderRequireds[id]) {
    // on draw
    childFlow?.emit?.("updateRenderRequired", [id, false]);
  }
});
/* end render flow */
const light = new RectAreaLight();

watchEffect(() => {
  light.color.set(props.color);
  light.intensity = props.strength;
  light.width = props.width;
  light.height = props.height;
});
</script>
