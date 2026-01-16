<script setup>
import * as Cesium from 'cesium';
import { onMounted, onUnmounted } from 'vue';

let viewer = null;

onMounted(() => {
  // 设置 Cesium Ion 访问令牌
  Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyMWExZmQwMS1jZTVhLTQ3YmQtYWE4Zi0zZTFiMTgyNmNlMjMiLCJpZCI6Mzc5NTU0LCJpYXQiOjE3Njg1NDkzMjV9.VlgFsSO52aKZCoV_EXZMYkK4X-OTDsEhpXbk2xomkqk';
  
  // 创建 Cesium Viewer
  viewer = new Cesium.Viewer('cesiumContainer', {
    terrain: Cesium.Terrain.fromWorldTerrain(),
    baseLayerPicker: false,
    vrButton: false,
    geocoder: false,
    homeButton: false,
    infoBox: false,
    sceneModePicker: false,
    selectionIndicator: false,
    timeline: false,
    animation: false,
  });
  
  // viewer.scene.globe.enableLighting = true;
});

onUnmounted(() => {
  if (viewer) {
    viewer.destroy();
  }
});
</script>

<template>
  <div id="cesiumContainer"></div>
</template>

<style scoped>

</style>
