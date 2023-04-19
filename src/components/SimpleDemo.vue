<script setup>
import * as spine from '@esotericsoftware/spine-webgl'
import { onMounted, ref, watch } from 'vue';


const getAssetsUrl = (name) => {
  return new URL(`/src/assets/${name}`, import.meta.url).href;
}

const animation = ref('')
const animations = ref([])

class App {

  constructor() {
    this.canvas = null;
    this.atlas = null;
    this.skeletonData = null;
    this.skeleton = null;
    this.state = null;
    this.selectedSkins = [];
    this.skinThumbnails = {};
    this.lastBounds = {};
  }

  loadAssets(canvas) {
    canvas.assetManager.AnimationState
    canvas.assetManager.loadTextureAtlas(getAssetsUrl("Skadi_Role.atlas"));
    canvas.assetManager.loadBinary(getAssetsUrl("Skadi_Role.skel"));
  }

  initialize(canvas) {
    this.canvas = canvas;
    let assetManager = canvas.assetManager;

    // Create the atlas 创建纹理集
    this.atlas = canvas.assetManager.require(getAssetsUrl("Skadi_Role.atlas"));
    let atlasLoader = new spine.AtlasAttachmentLoader(this.atlas);

    // Create the skeleton 创建骨架
    let skeletonBinary = new spine.SkeletonBinary(atlasLoader);
    this.skeletonData = skeletonBinary.readSkeletonData(assetManager.require(getAssetsUrl("Skadi_Role.skel")));
    this.skeleton = new spine.Skeleton(this.skeletonData);
    console.log(this.skeletonData)
    animations.value = this.skeletonData.animations.map(item => item.name)
    console.log(animations.value)

    // Create the animation state 创建动画状态
    let stateData = new spine.AnimationStateData(this.skeletonData);
    this.state = new spine.AnimationState(stateData);
    this.state.setAnimation(0, animations.value[0], true);
    animation.value = animations.value[0]
  }

  update(canvas, delta) {
    this.state.update(delta);
    this.state.apply(this.skeleton);
    this.skeleton.updateWorldTransform();

    let offset = new spine.Vector2(), size = new spine.Vector2();
    this.skeleton.getBounds(offset, size);
    this.lastBounds = { offset: offset, size: size };
  }

  render(canvas) {
    // Center and zoom the camera so the skeleton is
    // in full view irrespective of the page size.
    // 将相机居中并缩放，以便骨架
    // 在全视图中，而不管页面大小。
    // 获取渲染器
    let renderer = canvas.renderer;
    // 获取相机
    // 获取相机
    let camera = renderer.camera;
    // resize()方法是SpineCanvas的方法，它用于调整渲染器的大小
    renderer.resize(spine.ResizeMode.Expand);
    // 获取骨骼的边界
    let offset = this.lastBounds.offset, size = this.lastBounds.size;
    camera.position.x = offset.x + size.x / 2;
    camera.position.y = offset.y + size.y / 2;
    // 设置相机的缩放
    camera.zoom = size.x > size.y ? size.x / this.canvas.htmlCanvas.width * 1.1 : size.y / this.canvas.htmlCanvas.height * 1.1;
    camera.update();
    renderer.begin();
    renderer.drawSkeleton(this.skeleton, true);
    renderer.end();
  }
}

const instance = ref(null)

onMounted(() => {
  // Create the Spine canvas which runs the app
  instance.value = new App()
  new spine.SpineCanvas(document.getElementById("canvas"), {
    app: instance.value
  });
});

watch(animation, (val) => {
  if (val && instance.value) {
    instance.value.state.setAnimation(0, val, true);
    console.log(instance.value)
  }
})
</script>

<template>
  <div id="container">
    <canvas id="canvas"></canvas>
    <select v-model="animation">
      <option :value="item" v-for="(item, index) in animations" :key="index">{{ item }}</option>
    </select>
  </div>
</template>

<style scoped>
html,
body {
  margin: 0;
  padding: 0;
}

#container {
  display: flex;
  width: 80vh;
  height: 80vh;
  justify-content: center;
  align-items: center;
}

#canvas {
  width: 10vw;
  height: 10vw;
}
</style>
