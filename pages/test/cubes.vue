<template lang="pug">
div
  CoreViewport(v-model:flow="flow")
    template(#3d="{}")
      CoreGroup
        template(v-for="i in 5")
          CoreMeshCube(v-for="j in 5" v-bind={sx:.4,sy:.4,sz:.4} :dx="-i*.5" :dz="-j*.5")
        //- CoreLightPoint(:strength="30" v-bind="{dx:3,dy:2,dz:2}")
        CoreLightHemi(:strength="2")
</template>

<script setup lang="ts">

const flow = ref<ReturnType<typeof useRenderDataflow>>();

let running = true;
onUnmounted(() => {
  running = false;
});

const unsub = watch(flow, () => {
  if (flow.value) {
    unsub()
    onRenderFlowInjected(flow.value)
  }
});
onUnmounted(() => {
  removeClickables(flow.value!.provides.camera, flow.value!.provides.clickables)
})
const onRenderFlowInjected = (flow: ReturnType<typeof useRenderDataflow>) => {
  // console.log(JSON.stringify(flow.provides.clickables))
  addClickables(flow.provides.camera, flow.provides.clickables)

  const render = () => {
    running && requestAnimationFrame(render)
    flow.props.renderRequired = true;
  }
  render()
  // 毎フレーム実行
  watch(
    toRef(flow.props, "renderRequired"),
    (val) => {
      if (val) {
        // before render
        // console.log("before render")
      } else {
        // after render
        // console.log("render")
        flow.provides.renderer.render(toRaw(flow.provides.object3d),
          toRaw(flow.provides.camera))
        // console.log("after render")
      }
    }, { flush: "sync" })
}
</script>