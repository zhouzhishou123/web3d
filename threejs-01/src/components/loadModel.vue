<template>
  <div ref="containerRef" class="model-container"></div>
</template>

<script setup>
import * as THREE from 'three';
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';

const containerRef = ref(null);

let scene = null;
let camera = null;
let renderer = null;
let controls = null;
let mixer = null;
let model = null;
let animationId = null;
let clock = null;
let raycaster = null;

function init() {
  if (!containerRef.value) return;

  // 创建场景
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x222222);

  // 添加环境光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
  scene.add(ambientLight);

  // 添加方向光
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(5, 10, 5);
  scene.add(directionalLight);

  // 创建相机
  const width = containerRef.value.clientWidth;
  const height = containerRef.value.clientHeight;
  camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
  camera.position.set(0, 0, 5);

  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(width, height);
  renderer.setPixelRatio(window.devicePixelRatio);
  containerRef.value.appendChild(renderer.domElement);

  // 添加控制器
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;

  // 创建射线投射器用于点击检测
  raycaster = new THREE.Raycaster();

  // 添加鼠标点击事件监听
  renderer.domElement.addEventListener('click', onMouseClick);

  // 加载模型
  loadModel();

  // 创建时钟用于计算时间差
  clock = new THREE.Clock();

  // 处理窗口大小变化
  window.addEventListener('resize', handleResize);

  // 开始动画循环
  animate();
}

function loadModel() {
  // 创建并配置 DRACOLoader
  const dracoLoader = new DRACOLoader();
  // 使用 jsDelivr CDN 提供的 Draco 解码器（glTF 版本）
  // 这是最可靠的方式，适用于所有浏览器
  dracoLoader.setDecoderPath('https://cdn.jsdelivr.net/npm/three@0.153.0/examples/jsm/libs/draco/gltf/');
  
  // 备选方案：如果 CDN 不可用，可以取消下面的注释，并将 draco 文件复制到 public/draco/ 目录
  // dracoLoader.setDecoderPath('/draco/gltf/');
  
  const loader = new GLTFLoader();
  // 将 DRACOLoader 实例传递给 GLTFLoader
  loader.setDRACOLoader(dracoLoader);
  
  loader.load(
    '/models/LittlestTokyo.glb',
    (gltf) => {
      model = gltf.scene;
      scene.add(model);

      // 计算模型边界框，调整相机位置
      const box = new THREE.Box3().setFromObject(model);
      const center = box.getCenter(new THREE.Vector3());
      const size = box.getSize(new THREE.Vector3());
      
      // 将模型居中
      model.position.sub(center);
      
      // 调整相机位置以适应模型
      const maxDim = Math.max(size.x, size.y, size.z);
      const fov = camera.fov * (Math.PI / 180);
      let cameraZ = Math.abs(maxDim / 2 / Math.tan(fov / 2));
      cameraZ *= 1.5; // 稍微拉远一点
      camera.position.set(0, 0, cameraZ);
      camera.lookAt(0, 0, 0);
      controls.update();

      // 设置动画混合器
      if (gltf.animations && gltf.animations.length > 0) {
        mixer = new THREE.AnimationMixer(model);
        // 播放所有动画
        gltf.animations.forEach((clip) => {
          const action = mixer.clipAction(clip);
          action.play();
        });
        
        console.log(`已加载 ${gltf.animations.length} 个动画`);
      } else {
        console.warn('模型中没有找到动画');
      }
      
      // 模型加载完成后，可以释放 DRACOLoader 资源（可选）
      // dracoLoader.dispose();
    },
    (progress) => {
      const percentage = (progress.loaded / progress.total) * 100;
      console.log(`模型加载进度: ${percentage.toFixed(2)}%`);
    },
    (error) => {
      console.error('加载模型时出错:', error);
    }
  );
}

function animate() {
  animationId = requestAnimationFrame(animate);

  // 更新动画混合器（使用实际时间差）
  if (mixer && clock) {
    const delta = clock.getDelta();
    mixer.update(delta);
  }

  // 更新控制器
  if (controls) {
    controls.update();
  }

  // 渲染场景
  if (renderer && scene && camera) {
    renderer.render(scene, camera);
  }
}

function onMouseClick(event) {
  if (!raycaster || !camera || !model) return;

  // 获取鼠标在画布中的位置（相对于画布的坐标）
  const rect = renderer.domElement.getBoundingClientRect();
  const mouse = new THREE.Vector2();
  
  // 将鼠标坐标转换为标准化设备坐标（-1 到 1）
  mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

  // 更新射线投射器，从相机位置向鼠标方向发射射线
  raycaster.setFromCamera(mouse, camera);

  // 检测与模型相交的物体
  const intersects = raycaster.intersectObject(model, true);

  if (intersects.length > 0) {
    // 获取第一个被点击的物体
    const clickedObject = intersects[0].object;
    
    // 获取物体的名字（如果没有名字，尝试获取父对象的名字）
    let objectName = clickedObject.name || clickedObject.userData.name;
    
    // 如果物体没有名字，向上查找父对象的名字
    if (!objectName) {
      let parent = clickedObject.parent;
      while (parent && !objectName) {
        objectName = parent.name || parent.userData.name;
        parent = parent.parent;
      }
    }
    
    // 如果还是没有名字，使用对象的类型
    if (!objectName) {
      objectName = clickedObject.type || '未知物体';
    }
    
    // 打印物体名字
    console.log('点击的物体名称:', objectName);
    console.log('物体详细信息:', clickedObject);
  } else {
    console.log('未点击到任何物体');
  }
}

function handleResize() {
  if (!containerRef.value || !camera || !renderer) return;

  const width = containerRef.value.clientWidth;
  const height = containerRef.value.clientHeight;

  camera.aspect = width / height;
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);
}

function cleanup() {
  // 取消动画循环
  if (animationId) {
    cancelAnimationFrame(animationId);
  }

  // 移除事件监听
  window.removeEventListener('resize', handleResize);
  if (renderer && renderer.domElement) {
    renderer.domElement.removeEventListener('click', onMouseClick);
  }

  // 清理渲染器
  if (renderer) {
    renderer.dispose();
    if (containerRef.value && renderer.domElement) {
      containerRef.value.removeChild(renderer.domElement);
    }
  }

  // 清理控制器
  if (controls) {
    controls.dispose();
  }

  // 清理场景
  if (scene) {
    scene.traverse((object) => {
      if (object instanceof THREE.Mesh) {
        object.geometry?.dispose();
        if (Array.isArray(object.material)) {
          object.material.forEach((material) => material.dispose());
        } else {
          object.material?.dispose();
        }
      }
    });
  }
}

onMounted(() => {
  init();
});

onBeforeUnmount(() => {
  cleanup();
});
</script>

<style scoped>
.model-container {
  width: 100vw;
  height: 100vh;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>
