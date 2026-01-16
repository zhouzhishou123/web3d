<template>
    
</template>

<script setup>
import * as THREE from 'three';
import { onMounted } from 'vue';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

function init() {
        // 创建场景
    const scene = new THREE.Scene();
        // 创建相机
        const camera = new THREE.PerspectiveCamera( 
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
    );
    //添加坐标轴辅助器
    const axesHelper = new THREE.AxesHelper(5);
    scene.add(axesHelper);

    camera.position.z = 5; // 设置相机位置
    camera.lookAt(0, 0, 0);
    // 创建渲染器
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const controls = new OrbitControls( camera, renderer.domElement );
    // controls.autoRotate = true;
    // const loader = new GLTFLoader();
    // const gui = new GUI();

    // 创建几何体
    const geometry = new THREE.BoxGeometry( 1, 1, 1 );

    // 创建材质
    const material = new THREE.MeshBasicMaterial({ color: '#F5F5F5' });
    // 创建父元素的材质
    const parentMaterial = new THREE.MeshBasicMaterial({ color: '#F6B6B6' });

    // 创建一个父级对象
    const parentMesh = new THREE.Mesh( geometry, parentMaterial );

    // 沿着Z轴旋转45
    // parentMesh.rotation.z = Math.PI / 4;

    // 创建网格
    const mesh = new THREE.Mesh( geometry, material );
    mesh.position.x = 2;

    // 沿着Z轴旋转45
    // mesh.rotation.z = Math.PI / 4;
    // 将网格添加到父级对象中
    parentMesh.add(mesh);

    // 将网格添加到场景中
    scene.add(parentMesh);


    parentMesh.position.x = 2
    // 渲染
    renderer.render(scene, camera);


    window.addEventListener('resize', () => {
        renderer.setSize(window.innerWidth, window.innerHeight);
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.render(scene, camera);
    });
    
    // 动画
    function animate() {
        requestAnimationFrame(animate);
        controls.update(); // 更新控制器，使自动旋转生效
        parentMesh.rotation.x += 0.1;
        mesh.rotation.x += 0.1;
        // parentMesh.rotation.y += 0.01;
        renderer.render(scene, camera);
    }
    animate();
}


onMounted(() => {
    init();
});
</script>