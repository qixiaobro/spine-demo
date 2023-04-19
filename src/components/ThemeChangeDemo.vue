<script setup>
import * as spine from '@esotericsoftware/spine-webgl'
import { onMounted, ref, watch } from 'vue';


const getAssetsUrl = (name) => {
  return new URL(`/src/assets/${name}`, import.meta.url).href;
}

const animation = ref('walk')
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
    canvas.assetManager.loadTextureAtlas(getAssetsUrl("mix-and-match-pma.atlas"));
    canvas.assetManager.loadBinary(getAssetsUrl("mix-and-match-pma.skel"));
  }

  initialize(canvas) {
    this.canvas = canvas;
    let assetManager = canvas.assetManager;

    // Create the atlas 创建纹理集
    this.atlas = canvas.assetManager.require(getAssetsUrl("mix-and-match-pma.atlas"));
    let atlasLoader = new spine.AtlasAttachmentLoader(this.atlas);

    // Create the skeleton 创建骨架
    let skeletonBinary = new spine.SkeletonBinary(atlasLoader);
    this.skeletonData = skeletonBinary.readSkeletonData(assetManager.require(getAssetsUrl("mix-and-match-pma.skel")));
    this.skeleton = new spine.Skeleton(this.skeletonData);
    console.log(this.skeletonData)

    // Create the animation state 创建动画状态
    let stateData = new spine.AnimationStateData(this.skeletonData);
    this.state = new spine.AnimationState(stateData);
    this.state.setAnimation(0, "walk", true);
    animations.value = this.skeletonData.animations.map(item => item.name)

    // Create the user interface to selecting skins 创建皮肤选择界面
    this.createUI(canvas);

    // Create a default skin. 创建默认皮肤
    this.addSkin("skin-base");
    this.addSkin("nose/short");
    this.addSkin("eyelids/girly");
    this.addSkin("eyes/violet");
    this.addSkin("hair/brown");
    this.addSkin("clothes/hoodie-orange");
    this.addSkin("legs/pants-jeans");
    this.addSkin("accessories/bag");
    this.addSkin("accessories/hat-red-yellow");
  }

  addSkin(skinName) {
    // Don't add a skin that doesn't exist. 不要添加不存在的皮肤
    // Don't add the same skin twice. 不要添加相同的皮肤两次
    if (this.selectedSkins.indexOf(skinName) != -1) return;
    this.selectedSkins.push(skinName);
    // Update the thumbnail to show that it is selected. 更新缩略图以显示它已被选中
    let thumbnail = this.skinThumbnails[skinName];
    thumbnail.isSet = true;
    thumbnail.style.filter = "none";
    this.updateSkin();
  }

  removeSkin(skinName) {
    let index = this.selectedSkins.indexOf(skinName);
    if (index == -1) return;
    this.selectedSkins.splice(index, 1);
    let thumbnail = this.skinThumbnails[skinName];
    thumbnail.isSet = false;
    thumbnail.style.filter = "grayscale(1)";
    this.updateSkin();
  }

  updateSkin() {
    // Create a new skin from all the selected skins. 从所有选定的皮肤中创建一个新皮肤
    let newSkin = new spine.Skin("custom-skin");
    for (var skinName of this.selectedSkins) {
      newSkin.addSkin(this.skeletonData.findSkin(skinName));
    }
    this.skeleton.setSkin(newSkin);
    this.skeleton.setToSetupPose();
    this.skeleton.updateWorldTransform();

    // Calculate the bounds so we can center and zoom
    // the camera such that the skeleton is in full view.
    let offset = new spine.Vector2(), size = new spine.Vector2();
    this.skeleton.getBounds(offset, size);
    this.lastBounds = { offset: offset, size: size };
  }

  createUI(canvas) {
    const THUMBNAIL_SIZE = 100;
    let renderer = canvas.renderer;
    let camera = renderer.camera;

    // We'll reuse the webgl context used to render the skeleton for
    // thumbnail generation. We temporarily resize it to 300x300 pixels
    // Note: we passed `preserveDrawingBuffer: true` to the SpineCanvas
    // constructor. Without it, we could not fetch the pixel data from
    // the canvas after rendering.
    // 重用webgl上下文用于缩略图生成。我们临时将其调整为300x300像素
    // 注意：我们传递了`preserveDrawingBuffer：true`到SpineCanvas
    // 构造函数。如果没有它，我们将无法从
    // 渲染后的画布上获取像素数据。
    let oldWidth = canvas.htmlCanvas.width;
    let oldHeight = canvas.htmlCanvas.height;
    canvas.htmlCanvas.width = canvas.htmlCanvas.height = THUMBNAIL_SIZE;
    // canvas.gl.viewport是WebGL的一个API函数，它用于设置视口（viewport）的大小和位置。视口指的是WebGL绘图上下文中的一个矩形区域，它定义了可以显示渲染结果的屏幕区域。
    canvas.gl.viewport(0, 0, THUMBNAIL_SIZE, THUMBNAIL_SIZE);

    // For each skin, generate at thumbnail as follows 
    // 1. Set it on the skeleton
    // 2. Determine its bounds
    // 3. Center and scale it to the offscreen canvas and render it
    // 4. Fetch the rendered image from the canvas and store it.
    // 对于每个皮肤，生成一个缩略图如下
    // 1. 将其设置在骨架上
    // 2. 确定其边界
    // 3. 将其居中并缩放到离屏画布并渲染它
    // 4. 从画布中获取渲染的图像并将其存储。
    let images = [];

    for (var skin of this.skeletonData.skins) {
      // Skip the empty default skin 
      // 跳过空的默认皮肤
      if (skin.name === "default") continue;

      // Set the skin, then update the skeleton 
      // to the setup pose and calculate the world transforms
      // 设置皮肤，然后更新骨架
      // 到设置姿势并计算世界变换

      this.skeleton.setSkin(skin);
      this.skeleton.setToSetupPose();
      this.skeleton.updateWorldTransform();

      // Calculate the bounding box enclosing the skeleton.
      // 计算包围骨架的边界框。
      let offset = new spine.Vector2(), size = new spine.Vector2();
      // 获取骨骼的边界
      this.skeleton.getBounds(offset, size);

      // Position the renderer camera on the center of the bounds, and
      // set the zoom so the full skin is visible within the 300x300
      // rendering area. We leave 10% of empty space around a skin in the
      // thumbnail, hence the multiplication of 1.1 for the zoom factor.			
      // 将渲染器相机定位在边界的中心，并
      // 将缩放设置为全皮肤在300x300中可见
      // 渲染区域。我们在缩略图中留下10％的空间
      // 皮肤，因此缩放因子的乘法为1.1。
      camera.position.x = offset.x + size.x / 2;
      camera.position.y = offset.y + size.y / 2;
      camera.zoom = size.x > size.y ? size.x / THUMBNAIL_SIZE * 1.1 : size.y / THUMBNAIL_SIZE * 1.1;
      // 设置视口
      camera.setViewport(THUMBNAIL_SIZE, THUMBNAIL_SIZE);
      camera.update();

      // Clear the canvas and render the skeleton
      // 清除画布并渲染骨架
      canvas.clear(0.5, 0.5, 0.5, 1);
      // begin()和end()方法是SpineCanvas的方法，它们用于开始和结束渲染。
      renderer.begin();
      // drawSkeleton()方法是SpineCanvas的方法，它用于渲染骨架。
      renderer.drawSkeleton(this.skeleton, true);
      renderer.end();

      // Get the image data and convert it to an img element
      // 获取图像数据并将其转换为img元素
      let image = new Image();
      image.src = canvas.htmlCanvas.toDataURL();
      image.skinName = skin.name;
      image.isSet = false;
      image.style.filter = "grayscale(1)";

      // Set up a click listener that will add/remove the skin
      // when the thumbnail is clicked.
      // 设置单击侦听器，单击缩略图时将添加/删除皮肤。
      image.onclick = () => {
        if (image.isSet) this.removeSkin(image.skinName);
        else this.addSkin(image.skinName);
      }

      // Store the thumbail image in the list of all skin
      // thumbnails.
      // 将缩略图图像存储在所有皮肤缩略图的列表中。
      images.push(image);
      this.skinThumbnails[image.skinName] = image;
    }

    // Sort the list of skin thumbnails by name, so items
    // from the same folder end up next to each other.
    // 按名称对皮肤缩略图列表进行排序，以便来自同一文件夹的项目
    // 互相靠近。
    images.sort((a, b) => {
      return a.skinName > b.skinName ? 1 : -1;
    });

    // Add the thumbnails to the skins <div>
    // 将缩略图添加到skins <div>
    let skinsDiv = document.getElementById("skins");
    for (var thumbnail of images) {
      skinsDiv.appendChild(thumbnail);
    }

    // Restore the canvas size and camera of the renderer
    // 恢复渲染器的画布大小和相机
    canvas.htmlCanvas.width = oldWidth;
    canvas.htmlCanvas.height = oldHeight;
    // resize()方法是SpineCanvas的方法，它用于调整渲染器的大小
    renderer.resize(spine.ResizeMode.Expand);
    camera.position.x = 0;
    camera.position.y = 0;
    camera.zoom = 1;
  }

  update(canvas, delta) {
    this.state.update(delta);
    this.state.apply(this.skeleton);
    this.skeleton.updateWorldTransform();
  }

  render(canvas) {
    // Center and zoom the camera so the skeleton is
    // in full view irrespective of the page size.
    // 将相机居中并缩放，以便骨架
    // 在全视图中，而不管页面大小。
    // 获取渲染器
    let renderer = canvas.renderer;
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

    canvas.clear(0.2, 0.2, 0.2, 1);
    renderer.begin();
    renderer.drawSkeleton(this.skeleton, true);
    renderer.end();
  }
}

const instance = ref(null);
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
    <div id="skins"></div>
    <canvas id="canvas"></canvas>
    <select v-model="animation" style="height:20px">
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
  width: 80vw;
  height: 80vh;
}

#skins {
  width: 100px;
  flex-shrink: 0;
  overflow: scroll;
}

#canvas {
  width: calc(100% - 200px);
}
</style>
