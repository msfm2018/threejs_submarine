<template >
  <div class="f1">
    <button class="bt1" @click="handleBtn1Click" :class="{ active: selectedObj === '夜色潜航' }">夜色潜航</button>
    <div class="split"></div>
    <button class="bt2" @click="handleBtn2Click" :class="{ active: selectedObj === '胜利归来' }">胜利归来</button>
    <div class="split"></div>
    <button class="bt3" @click="handleBtn3Click" :class="{ active: selectedObj === '静默航行' }">静默航行</button>
  </div>

  <div class="info-box" v-if="cuurrentObj">
    {{ cuurrentObj }}
  </div>
  <div class="custom-text-container">
  <div  class="custom-text">杀</div>
  </div>
  <div class="custom-text-container1">
  <div  class="custom-text1">猎</div>
  </div>
  <div></div>
</template>

<script  setup>
import {  ref, onMounted, onUnmounted } from "vue";
import * as THREE from "three";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
import gsap from "gsap";

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

let hemisphereLight, scene;

let isSilent = false;
let clonemeshes = new THREE.Group();
let video = null; // 全局唯一
let actions = [];
let animationMixer;
let g = new THREE.Group();
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let particleGeometry = null;
let cuurrentObj = ref("");
let selectedObj = ref("");
let skyMaterial;
let vv;
let clock;
const isAnimating = ref(true);


let originalZ = 0;

    const modelMaterialList = [];
    const handleBtn1Click = () => {
      selectedObj.value='夜色潜航'
      mapChange("./111.mp4")
 
    };
const mapChange=(path)=>{
  if (video) {
    video.pause();
    video.src = ""; // 释放旧资源
  }
  video = document.createElement("video");
  video.src = path;
  video.muted = true;
  video.loop = true;
  video.autoplay = true;
  video.playsInline = true;
  video.crossOrigin = "anonymous";

  video.play().catch(e => console.warn("Video play failed:", e));

  const tex = new THREE.VideoTexture(video);
  tex.minFilter = THREE.LinearFilter;
  tex.magFilter = THREE.LinearFilter;
  tex.format = THREE.RGBFormat;

  skyMaterial.map = tex;
  skyMaterial.needsUpdate = true;
}

    const handleBtn2Click = () => {
      selectedObj.value='胜利归来'
      mapChange("./earth.mp4")
 
    };

    const handleBtn3Click = () => {
      selectedObj.value='静默航行'
      isSilent = !isSilent;
    };
    function createBubbleTexture() {
    const size = 64; // 贴图尺寸 (64x64 或 128x128)
    const canvas = document.createElement('canvas');
    canvas.width = size;
    canvas.height = size;
    const ctx = canvas.getContext('2d');
    
    // 创建一个径向渐变
    const gradient = ctx.createRadialGradient(
        size / 2, size / 2, 0, // 渐变中心
        size / 2, size / 2, size / 2 // 渐变半径
    );
    
    // 设置颜色停止点：中心白色不透明，边缘透明
    gradient.addColorStop(0, 'rgba(255, 255, 255, 1)'); // 中心
    gradient.addColorStop(0.5, 'rgba(255, 255, 255, 0.5)');
    gradient.addColorStop(1, 'rgba(255, 255, 255, 0)'); // 边缘透明

    // 绘制渐变
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, size, size);

    // 从 Canvas 创建 THREE.CanvasTexture
    const texture = new THREE.CanvasTexture(canvas);
    texture.needsUpdate = true;
    return texture;
}
    onMounted(() => {
      scene = new THREE.Scene();
      scene.background = new THREE.Color("#ccc");
      scene.environment = new THREE.Color("#ccc");
      const camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        1,
        2000
      );
      camera.position.set(1000, 128, 0);
      const renderer = new THREE.WebGLRenderer();
      document.body.appendChild(renderer.domElement);
      renderer.setSize(window.innerWidth, window.innerHeight);

      /**
       * 控制器
       */
      const controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.maxDistance = 10; // 最大缩放距离
      controls.minDistance = 1; // 最小缩放距离
      controls.minPolarAngle = 0; // 最小旋转角度
      controls.maxPolarAngle = (85 / 360) * 2 * Math.PI;

      scene.add(clonemeshes);

      const dracoLoader = new DRACOLoader();
      dracoLoader.setDecoderPath("/moduler/draco/gltf/");
      const loaderglb = new GLTFLoader();
      loaderglb.setDRACOLoader(dracoLoader);

      // 加载模型
      loaderglb.load(
        "./mode/cao.glb",
        function (gltf) {
          const model = gltf.scene;
          animationMixer = new THREE.AnimationMixer(model);
          gltf.animations.forEach((clip) => {
            const action = animationMixer.clipAction(clip);
            action.play();
            actions.push(action);
          });
          model.scale.set(0.35, 0.38, 0.2);
          model.position.set(0, 0.8, 0);
          model.rotation.x = Math.PI / 2;
           g.add(model);

          const controls = new OrbitControls(camera, renderer.domElement);
          controls.target.set(0, 0.5, 0);
          controls.update();
        },
        //加载时调用，显示进度条
        (xhr) => {
          //
        },

        //加载出错时调用
        function (error) {
          console.log("An error happened");
        }
      );

      // 创建粒子几何体
      particleGeometry = new THREE.BufferGeometry();
      var particleCount = 10000;

      // 为粒子几何体生成随机位置
      var positions = new Float32Array(particleCount * 3);
      // Modify the range for initial positions
      for (var i = 0; i < positions.length; i += 3) {
        positions[i] = (Math.random() - 0.5) * 20; // Adjust the range for X axis
        positions[i + 1] = (Math.random() - 0.5) * 20; // Adjust the range for Y axis
        positions[i + 2] = (Math.random() - 0.5) * 20; // Adjust the range for Z axis
      }

      particleGeometry.setAttribute(
        "position",
        new THREE.BufferAttribute(positions, 3)
      );
      //const txt = new THREE.TextureLoader().load("./disc.png");
      
    
      var sizes = new Float32Array(particleCount);
for (var i = 0; i < particleCount; i++) {
    // 随机大小，例如在 0.01 到 0.12 之间变化
    sizes[i] = THREE.MathUtils.randFloat(0.01, 0.12); 
}
particleGeometry.setAttribute("size", new THREE.BufferAttribute(sizes, 1));



      //txt.colorSpace = THREE.SRGBColorSpace;
      const txt = createBubbleTexture();

      
      var particleMaterial = new THREE.PointsMaterial({
        color: 0xADD8E6, // 浅蓝色
    size: 0.06, // 更小的尺寸
    opacity: 0.85, // 略高的透明度
    transparent: true,
    sizeAttenuation: true,
    map: txt, // 假设 txt 贴图是柔和的圆形
    blending: THREE.NormalBlending, // 更改为普通混合
    depthTest: true, // 保持开启以避免穿帮
    vertexColors: false, // 统一颜色
      });

      var colors = [];
for (var ii = 0; ii < particleCount; ii++) {
    var color = new THREE.Color();
    color.setHSL(Math.random(), 1.0, 0.5); // Random hue, full saturation, and medium lightness
    colors.push(color.r, color.g, color.b);
}

particleGeometry.setAttribute(
    "color",
    new THREE.BufferAttribute(new Float32Array(colors), 3)
);
      // particleMaterial.color.setHSL(1.0, 0.3, 0.7, THREE.SRGBColorSpace);

      // 创建粒子系统
      var particles = new THREE.Points(particleGeometry, particleMaterial);

      // 粒子初始速度向量
      var velocities = new Float32Array(particleCount * 3);
      for (let i = 0; i < velocities.length; i += 3) {
        // 设置初始速度，例如在 y 轴方向上的速度
    // X 轴的初始速度（小范围的随机偏移）
    velocities[i] = (Math.random() - 0.5) * 0.5; 
    // Y 轴的速度（向上浮动的速度，应该更大）
    velocities[i + 1] = 1 + Math.random() * 1; 
    // Z 轴的初始速度（小范围的随机偏移）
    velocities[i + 2] = (Math.random() - 0.5) * 0.5;
      }

      particleGeometry.setAttribute(
        "velocity",
        new THREE.BufferAttribute(velocities, 3)
      );

      // 将粒子系统添加到场景中
      scene.add(particles);

      const textureLoader = new THREE.TextureLoader();
      const skyGeometry = new THREE.SphereGeometry(400, 400, 400);
      const skyTexture = textureLoader.load("./disc.png"); // 天空纹理
      skyMaterial = new THREE.MeshBasicMaterial({
        map: skyTexture,
      });
      skyGeometry.scale(1, 1, -1);
      const sky = new THREE.Mesh(skyGeometry, skyMaterial);
      scene.add(sky);

      video = document.createElement("video"); // 创建视频
      // video.src = "./earth.mp4";
      video.src = "./111.mp4";
      video.muted = true;
      video.playsInline = true;
      video.autoplay = true;
      video.loop = true;
      video.play();
      let tex = new THREE.VideoTexture(video);
      tex.minFilter = THREE.LinearFilter;
      tex.magFilter = THREE.LinearFilter;
      tex.wrapS = tex.wrapT = THREE.ClampToEdgeWrapping;
      tex.format = THREE.RGBAFormat;

      // 更新材质
      skyMaterial.map = tex;
      skyMaterial.map.needsUpdate = true;

      clock = new THREE.Clock();
      /**
       * 加载模型
       */
      var loader = new FBXLoader();
      loader.load(
        "mode/2.fbx",
        function (object) {
          object.traverse(function (child) {
            // 判断是否为元素
            if (child.isMesh) {
              child.castShadow = true;
              child.receiveShadow = true;
              modelMaterialList.push(child);
            }
          });
          //缩放
          object.scale.set(0.1, 0.05, 0.1);
          //位置
          object.position.set(0, 0, 0);

          vv = new THREE.Group();
          vv.add(object);
          vv.add(g);
          g.position.copy(vv.position);
          g.position.z += 6.5;
          g.position.x += 6;
          g.position.y += -3.4;

          originalZ = 20;
          // 加入场景中
          scene.add(vv);
          /**
           *  光线
           */
           const ambientLight = new THREE.AmbientLight("#fff", 0.4); // 降低环境光强度
          scene.add(ambientLight);
          let directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
          directionalLight.position.set(-4, 8, 4);

          hemisphereLight = new THREE.HemisphereLight(0x4444FF, 0x000022, 0.6);
          hemisphereLight.position.set(0, 8, 0);

          scene.add(directionalLight);
          scene.add(hemisphereLight);

          render();
          addEventListener("mousemove", onDocumentMouseMove, false);
          addEventListener("mouseout", onDocumentMouseOut, false);
        },
        undefined,
        function (e) {
          console.error(e);
        }
      );

      let isCameraMoving = true;
      const randomizeCameraMovement = () => {
        isCameraMoving = !isCameraMoving;

        if (!isSilent) {
          gsap.to(camera.position, {
            x: (Math.random() - 0.5) * 20,
            y: 0,
            z: (Math.random() - 0.5) * 20,
            duration: 2,
          });
        }
      };

      setInterval(randomizeCameraMovement, 8000);

      const render = () => {
        controls.update();
        renderer.render(scene, camera);

        if (animationMixer && isAnimating.value) {
          animationMixer.update(clock.getDelta());
        }

        if (vv) {
          vv.position.z -= 0.001 + 0.05 * Math.random();
          if (vv.position.z < originalZ - 60) {
            vv.position.z = originalZ;
          }
        }
        // 获取粒子的位置数据
        var positions = particleGeometry.getAttribute("position");

        // 获取粒子速度数据
        var velocities = particleGeometry.getAttribute("velocity");
        const delta = clock.getDelta();
        // Update positions based on velocities
        for (var i = 0; i < positions.array.length; i += 3) {
          // // Update positions based on velocities
          positions.array[i] += velocities.array[i] * 0.005;
          positions.array[i + 1] += velocities.array[i + 1] * 0.005;
          positions.array[i + 2] += velocities.array[i + 2] * 0.005;
// 基础运动：位置 = 位置 + 速度 * 时间增量 * 速度系数
    // 速度系数 0.005 太小，建议使用 delta 并调整一个合理的系数，如 0.5
    
          // Check if positions are out of range and reset if needed
          if (
            positions.array[i] > 10 ||
            positions.array[i] < -10 ||
            positions.array[i + 1] > 10 ||
            positions.array[i + 1] < -10 ||
            positions.array[i + 2] > 10 ||
            positions.array[i + 2] < -10
          ) {
            positions.array[i] = (Math.random() - 0.5) * 20; // Adjust the range for X axis
            positions.array[i + 1] = -10;
            positions.array[i + 2] = (Math.random() - 0.5) * 20; // Adjust the range for Z axis


   

            velocities.array[i] = 0;
            velocities.array[i + 1] = 1;
            velocities.array[i + 2] = 0;
          }
        }

        // 通知Three.js更新缓冲区
        positions.needsUpdate = true;
        velocities.needsUpdate = true;

        requestAnimationFrame(render);
      };

      // 监听移入
      function onDocumentMouseMove(event) {
        event.preventDefault();
        mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

        raycaster.setFromCamera(mouse, camera);
        // 计算物体和射线的焦点
        const intersects = raycaster.intersectObjects(scene.children, true);

        // 获取 元素
        if (intersects.length) {
          for (let i = 0; i < intersects.length; i++) {
            if (intersects[i].object.type == "Mesh") {
              if (intersects[i].object.material.length) {
                let els = intersects[i].object.material;
                let len = els.length;
                for (let index = 0; index < len; index++) {
                  cuurrentObj.value = intersects[i].object.material[index].name;
                }
              } else {
                cuurrentObj.value = intersects[i].object.material.name;
              }
            }
          }
        } else {
          // 非模型移入时 清除信息状态
          cuurrentObj.value = "";
        }
      }

      // 监听移出
      function onDocumentMouseOut(event) {
        cuurrentObj.value = "";
      }
    });

    onUnmounted(() => {});
   
</script>

<style scoped>
.f1 {
  display: flex;
  justify-content: center;
  width: 100%;
  position: fixed;
  top: 20px;
}

.split {
  width: 1px;
  height: 30px;
  background-color: #ddd;
  margin: 0 1px;
  margin-top: 5px;
}

.bt1,
.bt2,
.bt3 {
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  background-color: #3498db;
  color: #fff;
  font-size: 16px;
  border-radius: 50%; /* Set border-radius to 50% for circular buttons */
  transition: background-color 0.3s ease;
}

.bt1.active,
.bt2.active,
.bt3.active {
  background-color: #d80e52; /* Change the background color on click */
}

.bt1:hover,
.bt2:hover {
  background-color: #2980b9;
}

/* .f1{
  display: flex;
  justify-content: center;
  width: 100%;
  position: fixed;
  top: 20px;
}
.split{
  width:2px;
  height: 10px;
}
.bt1 {

  top: 20px;
  border: none;
  
}
.bt2 {

  top: 40px;
  border: none;
} */
.info-box {
  width: 400px;
  height: 80px;
  background: rgb(212, 212, 212);
  position: absolute;
  right: 0;
}

.custom-text-container {
  position: fixed;
  top: 50%;
  transform: translateY(-50%);
  right: 0;
  text-align: center;
}

.custom-text {
  font-size: 74px;
  color: #e4d5d5;
  margin-top: 10px;
}

.custom-text-container1 {
  position: fixed;
  top: 38%;
  transform: translateY(-50%);
  right: 0;
  text-align: center;
}

.custom-text1 {
  font-size: 84px;
  color: #333;
  margin-top: 10px;
  font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif
}
</style>