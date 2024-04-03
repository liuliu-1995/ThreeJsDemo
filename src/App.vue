<template>
  <div id="app">
    <a id="modelChangeBtn" @click="changeModel">切换模型</a>
  </div>
</template>

<script>
import * as THREE from 'three';
import * as TWEEN from "@tweenjs/tween.js"
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

export default {
  name: 'App',
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      bufferGeometry: null,
      objIndex: 0,
      particalNum: 50000, // 设置粒子数量
      particalCompleteStep: 0,
      positions: [],
      modeSrcList: [
        '../static/gltfModel/phoenix_bird.glb',
        '../static/gltfModel/generic_police_car.glb',
        '../static/gltfModel/venice_mask.glb',
      ],
    }
  },
  mounted() {
    this.init();
    this.animate();
  },
  methods: {
    init() {
      // 创建场景
      this.scene = new THREE.Scene();
      //设置相机
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 10000);
      this.camera.position.z = 300;
      // 初始化渲染器
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.enablePan = false; //禁止右键拖拽
      // this.controls.enableRotate = false; //禁止旋转
      this.controls.minDistance = 200; // 设置相机最小距离
      this.controls.maxDistance = 250; // 设置相机最大距离
      this.controls.minZoom = 0.8; // 缩放
      this.controls.maxZoom  = 1.2; // 缩放
      this.controls.minPolarAngle = -Math.PI / 4; // 上下滚动最大角度
      this.controls.maxPolarAngle = Math.PI / 4; // 上下滚动最大角度
      this.controls.minAzimuthAngle = -Math.PI / 4; // 左右滚动最大角度
      this.controls.maxAzimuthAngle = Math.PI / 4;
      this.controls.update();
      document.body.appendChild(this.renderer.domElement);
       // 创建环境光
      const ambientLight = new THREE.AmbientLight(0xff0000, 0.1);
      this.scene.add(ambientLight);
      // 初始化粒子
      this.positions = new Float32Array(this.particalNum * 3);
      const scales = new Float32Array(this.particalNum);
      for(let i=0; i<this.particalNum; i++){
        //弄一堆坐标在-400~400之间的随时数字;
        let x = THREE.MathUtils.randFloatSpread(800);
        let y = THREE.MathUtils.randFloatSpread(800);
        let z = THREE.MathUtils.randFloatSpread(800);
        this.positions[i] = x;
        this.positions[i+1] = y;
        this.positions[i+2] = z;
        const size = THREE.MathUtils.randFloatSpread(12);
        scales[i] = size;
      }
      this.bufferGeometry = new THREE.BufferGeometry();
      this.bufferGeometry.setAttribute( 'position', new THREE.BufferAttribute(this.positions, 3));
      this.bufferGeometry.setAttribute( 'scale', new THREE.BufferAttribute(scales, 1));
      this.bufferGeometry.center();
      // 初始化贴图
      const textureLoader = new THREE.TextureLoader();
      textureLoader.crossOrigin = '';
      const mapDot = textureLoader.load('../static/textures/gradient.png');  // 圆点
      const material = new THREE.PointsMaterial({
        size: 1,
        sizeAttenuation: true,
        color: 0xffffff,
        transparent: true,
        opacity: 1,
        map: mapDot
      });
      const points = new THREE.Points(this.bufferGeometry, material);
      this.scene.add(points);
      // 加载3D模型
      this.loadObj(0);
      //事件监听
      window.addEventListener('resize', this.onWindowResize, false);
    },
    loadObj(index){
      const objSrc = this.modeSrcList[index]
      this.particalCompleteStep = 0;
      const loader = new GLTFLoader();
      loader.load(objSrc, object => {
        console.log('object', object);
        object.scene.traverse((child) => {
          if (child.isMesh) {
            if (this.objIndex === 1) {
              child.geometry.center();
              child.geometry.rotateX(-Math.PI / 6);
              child.geometry.rotateY(-Math.PI / 6);
              child.geometry.rotateZ(-Math.PI / 6);
            } else {
              child.geometry.center();
            }
          }
        });
        if (this.objIndex === 1) {
          object.scene.scale.set(1000, 1000, 1000)
        }
        const startPositions = this.bufferGeometry.getAttribute('position');
        const destPosition = this.combineBuffer(object.scene, 'position');
        this.tweenObj(startPositions, destPosition)
      });
    },
    combineBuffer(model, bufferName) {
      let count = 0;
      model.traverse(function (child) {
        if (child.isMesh) {
          const buffer = child.geometry.attributes[bufferName];
          count += buffer.array.length;
        }
      });
      const combined = new Float32Array(count);
      let offset = 0;
      model.traverse(function (child) {
        if (child.isMesh) {
          const buffer = child.geometry.attributes[bufferName];
          combined.set(buffer.array, offset);
          offset += buffer.array.length;
        }
      });
      return new THREE.BufferAttribute(combined, 3);
    },
    changeModel() {
      if (this.objIndex < this.modeSrcList.length - 1) {
        this.objIndex++;
      } else {
        this.objIndex = 0;
      }
      this.loadObj(this.objIndex);
    },
    tweenObj(startPositions, destPosition) {
      for(let i = 0; i< startPositions.count; i++) {
          const tween = new TWEEN.Tween(this.positions);
          const cur = i % destPosition.count;
          let percent = 1;
          if (this.objIndex === 0) {
            percent = 0.8;
          }
          if (this.objIndex === 1) {
            percent = 2;
          }
          if (this.objIndex === 2) {
            percent = 120;
          }
          tween.to({
              [i * 3]: destPosition.array[cur * 3] * percent,
              [i * 3 + 1]: destPosition.array[cur * 3 + 1] * percent,
              [i * 3 + 2]: destPosition.array[cur * 3 + 2] * percent,
          }, 1000);
          tween.easing(TWEEN.Easing.Exponential.InOut);
          tween.delay(1000);
          tween.onUpdate(() => {
            startPositions.needsUpdate = true;
          });
          tween.start();
      }
  },
  onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
  },
  render() {
      TWEEN.update();
      this.renderer.render(this.scene, this.camera);
  },
  animate() {
      this.render();
      this.controls.update();
      requestAnimationFrame(this.animate);
  },
},
}
</script>

<style>
body {
  background-color: #000000;
  margin: 0px;
  overflow: hidden;
  font-family: Monospace;
  font-size: 13px;
  text-align: center;
  font-weight: bold;
  text-align: center;
}
#modelChangeBtn{
  position: fixed;
  left: 50%;
  bottom: 20px;
  font-size:18px;
  color: #fff;
  cursor: pointer;
}
</style>
